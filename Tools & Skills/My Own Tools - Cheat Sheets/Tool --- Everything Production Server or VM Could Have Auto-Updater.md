
To be fair : It's my 6th version in a row and as you thought, I cut 50% of this script because you don't need to know everything about the structure I'm currently working within.

```powershell
# ==============================================================================================
# Cyberwatch findings remediation script - V3 (universal across VMs)
# Updates every item from the Cyberwatch "Technologie" report EXCEPT Oracle Java and 7-Zip.
#
# V3 changes vs V2:
#   - KB staging folder is now a SHARED network path (\\FILESERVER\Bases\Software_Staging\
#     KB_for_script) instead of a local per-machine folder, so you only drop each month's .msu
#     files ONE time for all VMs to pick up. IMPORTANT: if this script runs as SYSTEM via Task
#     Scheduler (like your other scripts), SYSTEM authenticates on the network using the
#     COMPUTER account (DOMAIN\MACHINENAME$), not your own user. The share/NTFS permissions on
#     FILESERVER must grant at least Read to "Domain Computers" (or explicitly to each target
#     machine account) for this to work, and Modify if you want the script to move processed
#     files into Installed\ on the share. If that's not possible, run the Task Scheduler job
#     under a dedicated service account instead of SYSTEM. The script detects and reports an
#     access/reachability problem clearly instead of failing silently.
#   - KB identification no longer trusts the filename. Trusting "the file is named right" was
#     exactly your concern, so instead the script reads the update's REAL identity from inside
#     the .msu itself via "dism.exe /Get-PackageInfo" (this is metadata baked into the package
#     by Microsoft, immune to renaming) and only falls back to parsing the filename if that
#     lookup fails for some reason (flagged as lower-confidence in the log when it happens).
#   - Wrong-architecture / wrong-edition files are not a silent risk: wusa.exe itself validates
#     applicability before touching anything and returns a specific "not applicable" exit code
#     (already handled below) rather than corrupting anything - so even a mis-dropped file just
#     gets skipped with a clear log line, never partially applied.
#   - Every downloaded installer (VC++, Defender platform, VMware Tools, TeamViewer, Firefox,
#     and the local copy of each KB) is now deleted immediately after use, in addition to the
#     end-of-script bulk cleanup - so temp files don't pile up even if the script errors out
#     partway through instead of only being cleaned up right at the very end.
#
# V2 changes vs V1:
#   - UNIVERSAL: every component now DETECTS whether it's actually installed on this machine
#     before doing anything. Not present -> skipped silently (INFO line in the summary), no
#     error. Same script can run unattended on any VM (DB server, app server, workstation...).
#   - KB (.msu) handling replaced entirely: no more hardcoded per-KB URLs (they change every
#     Patch Tuesday and differ between Windows 10 / Windows Server / .NET Framework KBs). The
#     script now scans a local STAGING FOLDER for .msu files you drop in yourself (downloaded
#     from https://www.catalog.update.microsoft.com for whatever Cyberwatch flags that month),
#     extracts the KB number from the filename, and installs whichever isn't already applied.
#     Already-installed or successfully-installed files are moved to a Installed\ subfolder so
#     the staging folder always shows what's still pending.
#   - VC++ redistributables: added the x64 variants (2008/2010/2012/2013) and the unified
#     2015-2022 (v14) family in both x86 and x64, since different VMs show different variants.
#     Each family is only touched if it is actually installed on that machine.
#   - No more immediate reboot logic: if a restart is needed, it is SCHEDULED for 20:00 the
#     same evening (shutdown.exe /r /t ...) instead of happening mid-day. Nothing reboots
#     immediately - the machine stays fully in prod during the day.
#   - TeamViewer Host (not the full client - corrected after seeing the real Cyberwatch report).
#     You said you'll trigger this part yourself outside of usage hours, so the logic (stop
#     service -> install -> restart service) is unchanged, just the branding/URL is fixed.
#   - PostgreSQL, MySQL Server, Veeam Backup & Replication: DETECTION + REPORTING ONLY for now
#     (see the big comment block before that section for why - short version: blindly
#     silent-upgrading a database engine or a backup product on an unattended schedule is a
#     different risk category than a VC++ redistributable, so I didn't want to guess at
#     that logic without checking with you first).
#
# Style/structure re-used from Update-DotNet-V3.ps1 (admin check, anti-overlap lock, transcript
# log, Add-Summary/Add-ErrorDetail helpers, Brevo email notification on error).
# ==============================================================================================

# ----------------------------------------------------------------------------------------------
# ADMIN RIGHTS CHECK (must run before anything else - installers below require elevation)
# ----------------------------------------------------------------------------------------------
$CurrentIdentity  = [Security.Principal.WindowsIdentity]::GetCurrent()
$CurrentPrincipal = New-Object Security.Principal.WindowsPrincipal($CurrentIdentity)
$IsAdmin          = $CurrentPrincipal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

if (-not $IsAdmin) {
    Write-Host "[CRITICAL ERROR] This script must be run as Administrator." -ForegroundColor Red
    Write-Host "                 Current user: $($CurrentIdentity.Name) (not elevated)." -ForegroundColor Red
    Write-Host "                 Re-launch PowerShell with 'Run as administrator' and try again." -ForegroundColor Yellow
    Write-Host "                 If running via Task Scheduler, ensure 'Run with highest privileges' is checked." -ForegroundColor Yellow
    if ([Environment]::UserInteractive -and $Host.Name -match "ConsoleHost") {
        Write-Host "`nPress any key to close this window..." -ForegroundColor Cyan
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit 1
}

# ----------------------------------------------------------------------------------------------
# ANTI-OVERLAP LOCK (prevents two instances running at once, e.g. after a Task Scheduler retry)
# ----------------------------------------------------------------------------------------------
$LockFile = "$env:TEMP\Update-CyberwatchSoftware-V2.lock"

if (Test-Path $LockFile) {
    $LockAge = (Get-Date) - (Get-Item $LockFile).LastWriteTime
    if ($LockAge.TotalHours -lt 2) {
        Write-Host "[CRITICAL ERROR] Another instance of this script appears to be running (lock file age: $([math]::Round($LockAge.TotalMinutes)) minutes)." -ForegroundColor Red
        Write-Host "                 Lock file: $LockFile" -ForegroundColor Red
        Write-Host "                 If you are sure no other instance is running (e.g. after a crash), delete this file manually and retry." -ForegroundColor Yellow
        exit 1
    } else {
        Write-Host "[INFO] Found a stale lock file (older than 2 hours), assuming previous run crashed. Continuing." -ForegroundColor Gray
        Remove-Item -Path $LockFile -Force -ErrorAction SilentlyContinue
    }
}
New-Item -Path $LockFile -ItemType File -Force | Out-Null

# ----------------------------------------------------------------------------------------------
# PERSISTENT LOG FILE (one timestamped .log file per run, same folder as the script itself).
# Old logs (60+ days) are cleaned up automatically.
# ----------------------------------------------------------------------------------------------
$LogFolder = "C:\Scripts\Logs"
if (-not (Test-Path $LogFolder)) {
    New-Item -Path $LogFolder -ItemType Directory -Force | Out-Null
}
$LogFile = Join-Path $LogFolder ("Update-CyberwatchSoftware-V2_{0}.log" -f (Get-Date -Format 'yyyy-MM-dd_HH-mm-ss'))
try {
    Start-Transcript -Path $LogFile -Append -ErrorAction Stop | Out-Null
} catch {
    Write-Host "[WARNING] Could not start log file at ${LogFile}: $_" -ForegroundColor Yellow
}

Get-ChildItem -Path $LogFolder -Filter "Update-CyberwatchSoftware-V2_*.log" -ErrorAction SilentlyContinue |
    Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-60) } |
    Remove-Item -Force -ErrorAction SilentlyContinue

# ----------------------------------------------------------------------------------------------
# CONFIGURATION
# ----------------------------------------------------------------------------------------------
$TempDownload = "$env:TEMP\CyberwatchUpdateTemp"

# --- KB staging folder (shared network path - see the SYSTEM/permissions note in the header
# comment block above). Drop the .msu files you download from
# https://www.catalog.update.microsoft.com in here (whatever Cyberwatch flags that month).
# The script identifies each file by reading its real metadata (dism.exe /Get-PackageInfo),
# NOT by trusting the filename - see the KB section below. Already-applied or
# successfully-installed files get moved to the Installed\ subfolder on the share automatically.
$KBStagingFolder = "\\FILESERVER\Bases\Software_Staging\KB_for_script"

# --- Windows Defender Antimalware Platform Update - official, stable Microsoft endpoint.
# Always serves the current platform package for the given architecture. Verified 20/07/2026.
$DefenderPlatformUrl = "https://definitionupdates.microsoft.com/packages?package=platform&arch=x64"

# --- VMware Tools - official "latest" package index (no vCenter ISO mount required).
# Source: packages.vmware.com, verified 20/07/2026.
$VMwareToolsIndexUrl = "https://packages.vmware.com/tools/releases/latest/windows/x64/"
$VMwareToolboxCmdPaths = @(
    "C:\Program Files\VMware\VMware Tools\VMwareToolboxCmd.exe",
    "C:\Program Files (x86)\VMware\VMware Tools\VMwareToolboxCmd.exe"
)

# --- Visual C++ Redistributables - fixed official Microsoft URLs, ALL variants seen across your
# Cyberwatch reports. Each one is only installed/updated if that exact family+arch is ALREADY
# present on the machine (never force-installs a family that isn't there).
# Source: https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist (page last
# updated 2026-03-09 per Microsoft, re-verified 20/07/2026).
$VCRedistTargets = @(
    [PSCustomObject]@{ Name = "2008 x86";      NamePattern = "*Visual C++ 2008*Redistributable*x86*";      Url = "https://download.microsoft.com/download/5/D/8/5D8C65CB-C849-4025-8E95-C3966CAFD8AE/vcredist_x86.exe";                          MinVersion = [version]"9.0.30729.5677";  SilentArgs = @("/q", "/norestart") }
    [PSCustomObject]@{ Name = "2008 x64";      NamePattern = "*Visual C++ 2008*Redistributable*x64*";      Url = "https://download.microsoft.com/download/5/D/8/5D8C65CB-C849-4025-8E95-C3966CAFD8AE/vcredist_x64.exe";                          MinVersion = [version]"9.0.30729.5677";  SilentArgs = @("/q", "/norestart") }
    [PSCustomObject]@{ Name = "2010 x86";      NamePattern = "*Visual C++ 2010*Redistributable*x86*";      Url = "https://download.microsoft.com/download/1/6/5/165255E7-1014-4D0A-B094-B6A430A6BFFC/vcredist_x86.exe";                          MinVersion = [version]"10.0.40219.325"; SilentArgs = @("/q", "/norestart") }
    [PSCustomObject]@{ Name = "2010 x64";      NamePattern = "*Visual C++ 2010*Redistributable*x64*";      Url = "https://download.microsoft.com/download/1/6/5/165255E7-1014-4D0A-B094-B6A430A6BFFC/vcredist_x64.exe";                          MinVersion = [version]"10.0.40219.325"; SilentArgs = @("/q", "/norestart") }
    [PSCustomObject]@{ Name = "2012 x86";      NamePattern = "*Visual C++ 2012*Redistributable*x86*";      Url = "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe";                     MinVersion = [version]"11.0.61030.0";   SilentArgs = @("/install", "/quiet", "/norestart") }
    [PSCustomObject]@{ Name = "2012 x64";      NamePattern = "*Visual C++ 2012*Redistributable*x64*";      Url = "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x64.exe";                     MinVersion = [version]"11.0.61030.0";   SilentArgs = @("/install", "/quiet", "/norestart") }
    [PSCustomObject]@{ Name = "2013 x86";      NamePattern = "*Visual C++ 2013*Redistributable*x86*";      Url = "https://aka.ms/highdpimfc2013x86enu";                                                                                          MinVersion = [version]"12.0.40664.0";   SilentArgs = @("/install", "/quiet", "/norestart") }
    [PSCustomObject]@{ Name = "2013 x64";      NamePattern = "*Visual C++ 2013*Redistributable*x64*";      Url = "https://aka.ms/highdpimfc2013x64enu";                                                                                          MinVersion = [version]"12.0.40664.0";   SilentArgs = @("/install", "/quiet", "/norestart") }
    [PSCustomObject]@{ Name = "2015-2022 x86"; NamePattern = "*Visual C++ 2015-2022*Redistributable*x86*"; Url = "https://aka.ms/vc14/vc_redist.x86.exe";                                                                                        MinVersion = [version]"0.0.0.0";        SilentArgs = @("/install", "/quiet", "/norestart") }  # always-latest permalink, no fixed target version
    [PSCustomObject]@{ Name = "2015-2022 x64"; NamePattern = "*Visual C++ 2015-2022*Redistributable*x64*"; Url = "https://aka.ms/vc14/vc_redist.x64.exe";                                                                                        MinVersion = [version]"0.0.0.0";        SilentArgs = @("/install", "/quiet", "/norestart") }  # always-latest permalink, no fixed target version
)

# --- TeamViewer HOST (confirmed from your Cyberwatch report - not the full client).
$TeamViewerUrl = "https://download.teamviewer.com/download/TeamViewer_Host_Setup_x64.exe"

# --- Mozilla Firefox - official stable "always latest" redirect. Adjust &lang= if your machines
# run an English UI instead of French.
$FirefoxUrl = "https://download.mozilla.org/?product=firefox-latest-ssl&os=win64&lang=fr"

# --- Scheduled restart time if any component needed one (24h format). Nothing reboots
# immediately - the machine stays in prod all day, and only reboots this evening if needed.
$ScheduledRestartHour   = 22
$ScheduledRestartMinute = 0

# --- EMAIL NOTIFICATION CONFIGURATION (used only if errors occur) ---
# Sent via Brevo's transactional email API (https://api.brevo.com).
$NotifyEmailTo   = "admin@yourdomain.local"
$NotifyEmailFrom = "sender@example.com"   # Must be a sender validated in your Brevo account
$BrevoApiKey     = "YOUR_BREVO_API_KEY_HERE"
$VMHostname      = $env:COMPUTERNAME

$VMIPAddresses = (Get-NetIPAddress -AddressFamily IPv4 -ErrorAction SilentlyContinue |
                  Where-Object { $_.IPAddress -notlike "127.*" -and $_.IPAddress -notlike "169.254.*" } |
                  Select-Object -ExpandProperty IPAddress) -join ", "
if ([string]::IsNullOrWhiteSpace($VMIPAddresses)) {
    $VMIPAddresses = "Unable to determine (no active IPv4 interface found)"
}

# Counters and lines for the final summary
$Global:SuccessCount    = 0
$Global:ErrorCount      = 0
$Global:InfoCount       = 0
$Global:SummaryLines    = @()
$Global:ErrorDetails    = @()
$Global:RestartRequired = $false

function Add-Summary {
    param([string]$Message, [string]$Type = "INFO")
    $Global:SummaryLines += "[$Type] $Message"
    switch ($Type) {
        "SUCCESS" { $Global:SuccessCount++ }
        "ERROR"   { $Global:ErrorCount++ }
        default   { $Global:InfoCount++ }
    }
}

function Add-ErrorDetail {
    param(
        [string]$Component,
        [string]$ExceptionMessage,
        [string]$Context = ""
    )
    $Global:ErrorDetails += [PSCustomObject]@{
        Component        = $Component
        ExceptionMessage = $ExceptionMessage
        Context          = $Context
        Timestamp        = (Get-Date -Format 'yyyy-MM-dd HH:mm:ss')
    }
}

function Get-InstalledApps {
    # Generic helper: returns every Uninstall-registry entry (both hives) whose DisplayName
    # matches the given wildcard pattern. Used everywhere below to decide "is this even
    # installed on this machine" before touching anything.
    param([Parameter(Mandatory)][string]$NamePattern)
    $Roots = @(
        "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*",
        "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*"
    )
    Get-ItemProperty -Path $Roots -ErrorAction SilentlyContinue |
        Where-Object { $_.DisplayName -like $NamePattern } |
        Select-Object DisplayName, DisplayVersion
}

function Send-FailureNotification {
    if ($Global:ErrorDetails.Count -eq 0) { return }

    if ($Global:ErrorCount -gt 0) { $SubjectTag = "$($Global:ErrorCount) error(s)" } else { $SubjectTag = "restart scheduled" }
    $Subject = "[Cyberwatch-Update] $SubjectTag on $VMHostname ($VMIPAddresses) - $(Get-Date -Format 'yyyy-MM-dd HH:mm')"

    $BodyLines = @()
    $BodyLines += "Automated Cyberwatch remediation script finished with issues."
    $BodyLines += "Host: $VMHostname"
    $BodyLines += "IP address(es): $VMIPAddresses"
    $BodyLines += "Run date: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
    $BodyLines += "Total: $Global:SuccessCount success | $Global:ErrorCount error(s) | $Global:InfoCount info"
    $BodyLines += ""
    $BodyLines += "--- ERROR DETAILS (raw, for debugging) ---"
    foreach ($Err in $Global:ErrorDetails) {
        $BodyLines += ""
        $BodyLines += "[$($Err.Timestamp)] Component: $($Err.Component)"
        $BodyLines += "Exception: $($Err.ExceptionMessage)"
        if ($Err.Context) { $BodyLines += "Context: $($Err.Context)" }
    }
    $BodyLines += ""
    $BodyLines += "--- FULL SUMMARY ---"
    $BodyLines += $Global:SummaryLines

    $PlainTextBody = $BodyLines -join "`n"
    Add-Type -AssemblyName System.Web -ErrorAction SilentlyContinue
    $HtmlBody = "<pre style='font-family:monospace; white-space:pre-wrap;'>$([System.Web.HttpUtility]::HtmlEncode($PlainTextBody))</pre>"

    $BrevoPayload = @{
        sender      = @{ email = $NotifyEmailFrom }
        to          = @(@{ email = $NotifyEmailTo })
        subject     = $Subject
        htmlContent = $HtmlBody
    } | ConvertTo-Json -Depth 5

    try {
        Invoke-RestMethod -Uri "https://api.brevo.com/v3/smtp/email" `
                           -Method Post `
                           -Headers @{ "api-key" = $BrevoApiKey; "Content-Type" = "application/json"; "Accept" = "application/json" } `
                           -Body $BrevoPayload `
                           -ErrorAction Stop | Out-Null
        Write-Host "[INFO] Failure notification email sent to $NotifyEmailTo via Brevo." -ForegroundColor Gray
    } catch {
        Write-Host "[WARNING] Could not send notification email via Brevo: $($_.Exception.Message)" -ForegroundColor Yellow
    }
}

Write-Host "=======================================================================" -ForegroundColor Cyan
Write-Host " Cyberwatch findings remediation script - V2 (universal) " -ForegroundColor Cyan
Write-Host " Run date and time: $(Get-Date -Format 'dd/MM/yyyy HH:mm:ss')" -ForegroundColor Cyan
Write-Host " Excluded on purpose: Oracle Java, 7-Zip" -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan

if (-not (Test-Path $TempDownload)) {
    New-Item -Path $TempDownload -ItemType Directory -Force | Out-Null
    Write-Host "[INFO] Temporary folder created: $TempDownload" -ForegroundColor Gray
}

# ----------------------------------------------------------------------------------------------
# PREREQUISITE CHECK: internet access
# ----------------------------------------------------------------------------------------------
Write-Host "`n[PREREQUISITE CHECK] Testing internet connection..." -ForegroundColor Blue
$InternetAvailable = $false
try {
    Invoke-WebRequest -Uri "https://www.microsoft.com" -Method Head -TimeoutSec 8 -UseBasicParsing -ErrorAction Stop | Out-Null
    $InternetAvailable = $true
    Write-Host "[OK] Connection established." -ForegroundColor Green
} catch {
    Write-Host "[CRITICAL ERROR] No internet access." -ForegroundColor Red
    Add-Summary "No internet access - only the local KB staging folder will be processed." "ERROR"
    Add-ErrorDetail -Component "Internet connectivity check" -ExceptionMessage $_.Exception.Message
}

# ------------------------------------------------------------------------------------------
# KB PACKAGES (.msu) FROM THE SHARED STAGING FOLDER - works offline, so this runs regardless
# of the internet check above (it's pure local/SMB file access, no internet needed).
# ------------------------------------------------------------------------------------------
function Get-MsuKBNumber {
    # Reads the update's REAL identity from inside the package via DISM, instead of trusting
    # the filename. Falls back to parsing the filename only if the DISM lookup fails, and says
    # so explicitly (lower confidence) when it does.
    param([Parameter(Mandatory)][string]$MsuPath)

    $Result = [PSCustomObject]@{ KBNumber = $null; Source = $null; Description = $null }

    try {
        $DismOutput = & dism.exe /Online /Get-PackageInfo /PackagePath:"$MsuPath" 2>&1
        $IdentityLine = $DismOutput | Where-Object { $_ -match 'Package Identity' }
        $DescriptionLine = $DismOutput | Where-Object { $_ -match '^Description' } | Select-Object -First 1
        if ($IdentityLine -and $IdentityLine -match '(?i)kb(\d+)') {
            $Result.KBNumber    = "KB$($Matches[1])"
            $Result.Source      = "DISM package metadata (reliable)"
            $Result.Description = if ($DescriptionLine) { ($DescriptionLine -split ':', 2)[1].Trim() } else { $null }
            return $Result
        }
    } catch { }

    # Fallback: filename-based, explicitly flagged as lower confidence.
    $KBMatch = [regex]::Match((Split-Path -Leaf $MsuPath), '(?i)kb(\d+)')
    if ($KBMatch.Success) {
        $Result.KBNumber = "KB$($KBMatch.Groups[1].Value)"
        $Result.Source    = "filename only (DISM lookup failed - LOWER CONFIDENCE, double-check this one)"
    }
    return $Result
}

Write-Host "`n[KB STAGING FOLDER] $KBStagingFolder" -ForegroundColor Blue

$KBFolderReachable = $false
try {
    if (Test-Path -Path $KBStagingFolder -ErrorAction Stop) {
        $KBFolderReachable = $true
    } else {
        New-Item -Path $KBStagingFolder -ItemType Directory -Force -ErrorAction Stop | Out-Null
        $KBFolderReachable = $true
        Write-Host "  [INFO] Folder was empty/missing, created it." -ForegroundColor Gray
    }
} catch {
    Write-Host "  [ERROR] Cannot reach or create '$KBStagingFolder'." -ForegroundColor Red
    Write-Host "          If this script runs as SYSTEM via Task Scheduler, this is almost always a share/NTFS" -ForegroundColor Yellow
    Write-Host "          permission issue: SYSTEM authenticates on the network as the COMPUTER account" -ForegroundColor Yellow
    Write-Host "          (DOMAIN\$VMHostname`$), not as you. Grant that account (or 'Domain Computers') at" -ForegroundColor Yellow
    Write-Host "          least Read on the share, or run this task under a dedicated service account instead." -ForegroundColor Yellow
    Add-Summary "KB staging folder unreachable: $_" "ERROR"
    Add-ErrorDetail -Component "KB staging folder access" -ExceptionMessage $_.Exception.Message -Context "Path: $KBStagingFolder (likely SYSTEM/computer-account permission issue if run via Task Scheduler)"
}

if ($KBFolderReachable) {
    $InstalledSubfolder = Join-Path $KBStagingFolder "Installed"
    if (-not (Test-Path $InstalledSubfolder)) {
        try { New-Item -Path $InstalledSubfolder -ItemType Directory -Force -ErrorAction Stop | Out-Null }
        catch { Write-Host "  [WARNING] Could not create Installed\ subfolder on the share (read-only access?): $_" -ForegroundColor Yellow }
    }

    $MsuFiles = Get-ChildItem -Path $KBStagingFolder -Filter "*.msu" -File -ErrorAction SilentlyContinue
    if (-not $MsuFiles) {
        Write-Host "  [INFO] No .msu file waiting in the staging folder - nothing to do." -ForegroundColor Gray
        Add-Summary "KB staging folder empty, nothing to install." "INFO"
    }

    foreach ($MsuFile in $MsuFiles) {
        $Identity = Get-MsuKBNumber -MsuPath $MsuFile.FullName
        if (-not $Identity.KBNumber) {
            Write-Host "  [WARNING] Could not identify the KB number for '$($MsuFile.Name)' (neither DISM nor filename worked) - skipped." -ForegroundColor Yellow
            Add-Summary "Could not identify KB number for '$($MsuFile.Name)', skipped." "ERROR"
            continue
        }
        $KBNumber = $Identity.KBNumber
        Write-Host "  [KB CHECK] $KBNumber ($($MsuFile.Name))" -ForegroundColor Blue
        Write-Host "    Identified via: $($Identity.Source)" -ForegroundColor Gray
        if ($Identity.Description) { Write-Host "    Title: $($Identity.Description)" -ForegroundColor Gray }

        if (Get-HotFix -Id $KBNumber -ErrorAction SilentlyContinue) {
            Write-Host "    [OK] $KBNumber already installed - archiving the file." -ForegroundColor Green
            Add-Summary "$KBNumber already installed." "INFO"
            Move-Item -Path $MsuFile.FullName -Destination $InstalledSubfolder -Force -ErrorAction SilentlyContinue
            continue
        }

        # Copy locally before installing: more reliable than running wusa.exe straight off a
        # network share (avoids partial application on a flaky link), and keeps the network
        # copy untouched until we know the result.
        $LocalMsu = Join-Path $TempDownload $MsuFile.Name
        try {
            Copy-Item -Path $MsuFile.FullName -Destination $LocalMsu -Force -ErrorAction Stop

            Write-Host "    -> Installing via wusa.exe /quiet /norestart (from local copy)..." -ForegroundColor Yellow
            $Process = Start-Process -FilePath "wusa.exe" -ArgumentList "`"$LocalMsu`" /quiet /norestart" -Wait -NoNewWindow -PassThru -ErrorAction Stop

            switch ($Process.ExitCode) {
                0 {
                    Write-Host "    [SUCCESS] $KBNumber installed." -ForegroundColor Green
                    Add-Summary "$KBNumber installed successfully." "SUCCESS"
                    Move-Item -Path $MsuFile.FullName -Destination $InstalledSubfolder -Force -ErrorAction SilentlyContinue
                }
                3010 {
                    Write-Host "    [SUCCESS] $KBNumber installed, restart required." -ForegroundColor Green
                    Add-Summary "$KBNumber installed, restart required." "SUCCESS"
                    $Global:RestartRequired = $true
                    Move-Item -Path $MsuFile.FullName -Destination $InstalledSubfolder -Force -ErrorAction SilentlyContinue
                }
                2359302 {
                    Write-Host "    [INFO] Not applicable to this system (code 2359302 - often means already installed, but can also mean wrong architecture/build/edition)." -ForegroundColor Gray
                    Add-Summary "$KBNumber : not applicable per wusa (check architecture/build if unexpected)." "INFO"
                    Move-Item -Path $MsuFile.FullName -Destination $InstalledSubfolder -Force -ErrorAction SilentlyContinue
                }
                default {
                    Write-Host "    [ERROR] wusa.exe returned unexpected exit code $($Process.ExitCode). File kept in staging folder for retry." -ForegroundColor Red
                    Add-Summary "$KBNumber installation failed (wusa exit code $($Process.ExitCode))." "ERROR"
                    Add-ErrorDetail -Component "$KBNumber install" -ExceptionMessage "wusa.exe exit code $($Process.ExitCode)" -Context "File: $($MsuFile.FullName)"
                }
            }
        } catch {
            Write-Host "    [ERROR] Failed to install ${KBNumber}: $_" -ForegroundColor Red
            Add-Summary "Error while installing ${KBNumber}: $_" "ERROR"
            Add-ErrorDetail -Component "$KBNumber install" -ExceptionMessage $_.Exception.Message -Context "File: $($MsuFile.FullName)"
        } finally {
            # Immediate cleanup of the local copy, don't wait for the end-of-script bulk cleanup.
            Remove-Item -Path $LocalMsu -Force -ErrorAction SilentlyContinue
        }
    }
}
Write-Host "  Note: the CPE entries (microsoft:windows_10 / microsoft:windows_server_2022) are" -ForegroundColor Gray
Write-Host "  just the OS build number - they resolve automatically once the matching cumulative" -ForegroundColor Gray
Write-Host "  KB above is installed. No separate action needed for those." -ForegroundColor Gray

if ($InternetAvailable) {

    # ------------------------------------------------------------------------------------------
    # WINDOWS DEFENDER ANTIMALWARE PLATFORM UPDATE (skip if Defender isn't the active AV)
    # ------------------------------------------------------------------------------------------
    Write-Host "`n[DEFENDER PLATFORM UPDATE]" -ForegroundColor Blue
    if (-not (Get-Service -Name "WinDefend" -ErrorAction SilentlyContinue)) {
        Write-Host "  [SKIP] Windows Defender service not found on this machine." -ForegroundColor Gray
        Add-Summary "Defender platform update skipped: WinDefend service not present." "INFO"
    } else {
        try {
            $BeforeVersion = $null
            try { $BeforeVersion = (Get-MpComputerStatus -ErrorAction Stop).AMProductVersion } catch { }
            if ($BeforeVersion) { Write-Host "  Current platform version: $BeforeVersion" -ForegroundColor Gray }

            $PlatformFile = Join-Path $TempDownload "DefenderPlatformUpdate.exe"
            Write-Host "  -> Downloading current platform package from Microsoft..." -ForegroundColor Yellow
            Invoke-WebRequest -Uri $DefenderPlatformUrl -OutFile $PlatformFile -UseBasicParsing -ErrorAction Stop
            Write-Host "  [OK] Downloaded ($([math]::Round((Get-Item $PlatformFile).Length / 1MB, 1)) MB)." -ForegroundColor Green

            Write-Host "  -> Running the platform update package (self-installing, no switches needed)..." -ForegroundColor Yellow
            $Process = Start-Process -FilePath $PlatformFile -Wait -NoNewWindow -PassThru -ErrorAction Stop

            if ($Process.ExitCode -eq 0) {
                $AfterVersion = $null
                try { $AfterVersion = (Get-MpComputerStatus -ErrorAction Stop).AMProductVersion } catch { }
                Write-Host "  [SUCCESS] Platform update executed. Version now: $AfterVersion" -ForegroundColor Green
                Add-Summary "Defender platform updated (was $BeforeVersion, now $AfterVersion)." "SUCCESS"
            } else {
                Write-Host "  [WARNING] Platform update package returned exit code $($Process.ExitCode)." -ForegroundColor Yellow
                Add-Summary "Defender platform update returned exit code $($Process.ExitCode)." "ERROR"
                Add-ErrorDetail -Component "Defender platform update" -ExceptionMessage "Exit code $($Process.ExitCode)" -Context "File: $PlatformFile"
            }
        } catch {
            Write-Host "  [ERROR] Failed to update Defender platform: $_" -ForegroundColor Red
            Add-Summary "Error while updating Defender platform: $_" "ERROR"
            Add-ErrorDetail -Component "Defender platform update" -ExceptionMessage $_.Exception.Message -Context "Url: $DefenderPlatformUrl"
        } finally {
            Remove-Item -Path $PlatformFile -Force -ErrorAction SilentlyContinue
        }
    }

    # ------------------------------------------------------------------------------------------
    # VISUAL C++ REDISTRIBUTABLES - all variants, each one skipped if not installed on this VM
    # ------------------------------------------------------------------------------------------
    foreach ($VC in $VCRedistTargets) {
        Write-Host "`n[VC++ $($VC.Name) CHECK]" -ForegroundColor Blue
        $Found = Get-InstalledApps -NamePattern $VC.NamePattern
        if (-not $Found) {
            Write-Host "  [SKIP] Not installed on this machine." -ForegroundColor Gray
            continue
        }

        $CurrentVersion = [version]"0.0.0.0"
        foreach ($Entry in $Found) {
            if ($Entry.DisplayVersion -match '(\d+(\.\d+){1,3})') {
                $V = [version]$Matches[1]
                if ($V -gt $CurrentVersion) { $CurrentVersion = $V }
            }
        }
        Write-Host "  Installed: $CurrentVersion" -ForegroundColor Gray

        if ($CurrentVersion -ge $VC.MinVersion -and $VC.MinVersion -ne [version]"0.0.0.0") {
            Write-Host "  [OK] Already up to date (>= $($VC.MinVersion))." -ForegroundColor Green
            Add-Summary "VC++ $($VC.Name) already up to date." "INFO"
            continue
        }

        try {
            $VCFile = Join-Path $TempDownload "vcredist_$($VC.Name -replace ' ', '_').exe"
            Write-Host "  -> Downloading..." -ForegroundColor Yellow
            Invoke-WebRequest -Uri $VC.Url -OutFile $VCFile -UseBasicParsing -ErrorAction Stop
            Write-Host "  [OK] Downloaded ($([math]::Round((Get-Item $VCFile).Length / 1MB, 1)) MB)." -ForegroundColor Green

            Write-Host "  -> Installing silently ($($VC.SilentArgs -join ' '))..." -ForegroundColor Yellow
            $Process = Start-Process -FilePath $VCFile -ArgumentList $VC.SilentArgs -Wait -NoNewWindow -PassThru -ErrorAction Stop

            if ($Process.ExitCode -eq 0 -or $Process.ExitCode -eq 3010) {
                Write-Host "  [SUCCESS] VC++ $($VC.Name) installed/updated." -ForegroundColor Green
                Add-Summary "VC++ $($VC.Name) installed/updated." "SUCCESS"
                if ($Process.ExitCode -eq 3010) { $Global:RestartRequired = $true }
            } else {
                Write-Host "  [ERROR] Installer returned exit code $($Process.ExitCode)." -ForegroundColor Red
                Add-Summary "VC++ $($VC.Name) installation failed (exit code $($Process.ExitCode))." "ERROR"
                Add-ErrorDetail -Component "VC++ $($VC.Name) install" -ExceptionMessage "Exit code $($Process.ExitCode)" -Context "File: $VCFile"
            }
        } catch {
            Write-Host "  [ERROR] Failed to download/install VC++ $($VC.Name): $_" -ForegroundColor Red
            Add-Summary "Error while processing VC++ $($VC.Name): $_" "ERROR"
            Add-ErrorDetail -Component "VC++ $($VC.Name) download/install" -ExceptionMessage $_.Exception.Message -Context "Url: $($VC.Url)"
        } finally {
            Remove-Item -Path $VCFile -Force -ErrorAction SilentlyContinue
        }
    }

    # ------------------------------------------------------------------------------------------
    # VMWARE TOOLS (skip if this VM doesn't have it)
    # ------------------------------------------------------------------------------------------
    Write-Host "`n[VMWARE TOOLS CHECK]" -ForegroundColor Blue
    $ToolboxCmd = $VMwareToolboxCmdPaths | Where-Object { Test-Path $_ } | Select-Object -First 1
    if (-not $ToolboxCmd) {
        Write-Host "  [SKIP] VMware Tools not found on this machine." -ForegroundColor Gray
        Add-Summary "VMware Tools skipped: not installed on this machine." "INFO"
    } else {
        try {
            $InstalledVersion = $null
            $RawVersion = & $ToolboxCmd -v 2>$null
            if ($RawVersion -match '(\d+\.\d+\.\d+)') { $InstalledVersion = [version]$Matches[1] }
            if ($InstalledVersion) { Write-Host "  Installed version: $InstalledVersion" -ForegroundColor Gray }

            Write-Host "  -> Checking latest version at $VMwareToolsIndexUrl..." -ForegroundColor Yellow
            $IndexPage = Invoke-WebRequest -Uri $VMwareToolsIndexUrl -UseBasicParsing -ErrorAction Stop
            $Match = [regex]::Match($IndexPage.Content, 'VMware-tools-([\d\.]+)-\d+-x64\.exe')

            if (-not $Match.Success) {
                Write-Host "  [ERROR] Could not find the installer filename on the VMware index page." -ForegroundColor Red
                Add-Summary "VMware Tools: could not parse latest version from $VMwareToolsIndexUrl." "ERROR"
                Add-ErrorDetail -Component "VMware Tools version lookup" -ExceptionMessage "Regex did not match" -Context "Url: $VMwareToolsIndexUrl"
            } else {
                $LatestFileName = $Match.Value
                $LatestVersion  = [version]$Match.Groups[1].Value
                $LatestUrl      = "$VMwareToolsIndexUrl$LatestFileName"
                Write-Host "  Latest available version: $LatestVersion ($LatestFileName)" -ForegroundColor Gray

                if ($InstalledVersion -and $InstalledVersion -ge $LatestVersion) {
                    Write-Host "  [OK] Already up to date." -ForegroundColor Green
                    Add-Summary "VMware Tools already up to date ($InstalledVersion)." "INFO"
                } else {
                    $VMwareFile = Join-Path $TempDownload $LatestFileName
                    Write-Host "  -> Downloading $LatestFileName..." -ForegroundColor Yellow
                    Invoke-WebRequest -Uri $LatestUrl -OutFile $VMwareFile -UseBasicParsing -ErrorAction Stop
                    Write-Host "  [OK] Downloaded ($([math]::Round((Get-Item $VMwareFile).Length / 1MB, 1)) MB)." -ForegroundColor Green

                    Write-Host "  -> Installing silently (/S /v`"/qn REBOOT=R`")..." -ForegroundColor Yellow
                    $Process = Start-Process -FilePath $VMwareFile -ArgumentList @('/S', '/v"/qn REBOOT=R"') -Wait -NoNewWindow -PassThru -ErrorAction Stop

                    if ($Process.ExitCode -eq 0 -or $Process.ExitCode -eq 3010) {
                        Write-Host "  [SUCCESS] VMware Tools updated to $LatestVersion." -ForegroundColor Green
                        Add-Summary "VMware Tools updated to $LatestVersion." "SUCCESS"
                        if ($Process.ExitCode -eq 3010) { $Global:RestartRequired = $true }
                    } else {
                        Write-Host "  [ERROR] Installer returned exit code $($Process.ExitCode)." -ForegroundColor Red
                        Add-Summary "VMware Tools installation failed (exit code $($Process.ExitCode))." "ERROR"
                        Add-ErrorDetail -Component "VMware Tools install" -ExceptionMessage "Exit code $($Process.ExitCode)" -Context "File: $VMwareFile"
                    }
                }
            }
        } catch {
            Write-Host "  [ERROR] Failed to check/update VMware Tools: $_" -ForegroundColor Red
            Add-Summary "Error while processing VMware Tools: $_" "ERROR"
            Add-ErrorDetail -Component "VMware Tools" -ExceptionMessage $_.Exception.Message -Context "IndexUrl: $VMwareToolsIndexUrl"
        } finally {
            if ($VMwareFile) { Remove-Item -Path $VMwareFile -Force -ErrorAction SilentlyContinue }
        }
    }

    # ------------------------------------------------------------------------------------------
    # TEAMVIEWER HOST (skip if not installed)
    # You said you'll trigger this part yourself outside of usage hours - logic unchanged, the
    # service stop/restart below WILL interrupt an active remote session.
    # ------------------------------------------------------------------------------------------
    Write-Host "`n[TEAMVIEWER HOST CHECK]" -ForegroundColor Blue
    $TVFound = Get-InstalledApps -NamePattern "*TeamViewer*"
    if (-not $TVFound) {
        Write-Host "  [SKIP] TeamViewer not found on this machine." -ForegroundColor Gray
        Add-Summary "TeamViewer skipped: not installed on this machine." "INFO"
    } else {
        try {
            $InstalledVersion = ($TVFound | Sort-Object { [version]$_.DisplayVersion } -Descending | Select-Object -First 1).DisplayVersion
            if ($InstalledVersion) { Write-Host "  Installed version: $InstalledVersion" -ForegroundColor Gray }

            $TVFile = Join-Path $TempDownload "TeamViewer_Host_Setup_x64.exe"
            Write-Host "  -> Downloading latest TeamViewer Host from teamviewer.com..." -ForegroundColor Yellow
            Invoke-WebRequest -Uri $TeamViewerUrl -OutFile $TVFile -UseBasicParsing -ErrorAction Stop
            $TVFileInfo = Get-Item $TVFile
            $SizeMB = [math]::Round($TVFileInfo.Length / 1MB, 1)
            # ProductVersion is occasionally blank on this installer's version resource - fall
            # back to FileVersion, which is always populated.
            $LatestVersion = $TVFileInfo.VersionInfo.ProductVersion
            if ([string]::IsNullOrWhiteSpace($LatestVersion)) { $LatestVersion = $TVFileInfo.VersionInfo.FileVersion }
            Write-Host "  [OK] Downloaded ($SizeMB MB), version $LatestVersion." -ForegroundColor Green

            if ($InstalledVersion -and $LatestVersion -and ([version]$InstalledVersion -ge [version]$LatestVersion)) {
                Write-Host "  [OK] Already up to date." -ForegroundColor Green
                Add-Summary "TeamViewer Host already up to date ($InstalledVersion)." "INFO"
            } else {
                Write-Host "  -> Stopping TeamViewer service before update (will interrupt active sessions)..." -ForegroundColor Yellow
                Stop-Service -Name "TeamViewer" -Force -ErrorAction SilentlyContinue

                Write-Host "  -> Installing silently (/S /ACCEPTEULA=1)..." -ForegroundColor Yellow
                $Process = Start-Process -FilePath $TVFile -ArgumentList @('/S', '/ACCEPTEULA=1') -Wait -NoNewWindow -PassThru -ErrorAction Stop

                Start-Service -Name "TeamViewer" -ErrorAction SilentlyContinue

                if ($Process.ExitCode -eq 0 -or $Process.ExitCode -eq 3010) {
                    $VersionLabel = if ($LatestVersion) { $LatestVersion } else { "latest" }
                    Write-Host "  [SUCCESS] TeamViewer Host updated to $VersionLabel." -ForegroundColor Green
                    Add-Summary "TeamViewer Host updated to $VersionLabel (was $InstalledVersion)." "SUCCESS"
                    if ($Process.ExitCode -eq 3010) { $Global:RestartRequired = $true }
                } else {
                    Write-Host "  [ERROR] Installer returned exit code $($Process.ExitCode)." -ForegroundColor Red
                    Add-Summary "TeamViewer Host installation failed (exit code $($Process.ExitCode))." "ERROR"
                    Add-ErrorDetail -Component "TeamViewer Host install" -ExceptionMessage "Exit code $($Process.ExitCode)" -Context "File: $TVFile"
                }
            }
        } catch {
            Write-Host "  [ERROR] Failed to check/update TeamViewer Host: $_" -ForegroundColor Red
            Add-Summary "Error while processing TeamViewer Host: $_" "ERROR"
            Add-ErrorDetail -Component "TeamViewer Host" -ExceptionMessage $_.Exception.Message -Context "Url: $TeamViewerUrl"
            Start-Service -Name "TeamViewer" -ErrorAction SilentlyContinue
        } finally {
            if ($TVFile) { Remove-Item -Path $TVFile -Force -ErrorAction SilentlyContinue }
        }
    }

    # ------------------------------------------------------------------------------------------
    # MOZILLA FIREFOX (skip if not installed)
    # ------------------------------------------------------------------------------------------
    Write-Host "`n[FIREFOX CHECK]" -ForegroundColor Blue
    $FirefoxFound = Get-InstalledApps -NamePattern "Mozilla Firefox*"
    if (-not $FirefoxFound) {
        Write-Host "  [SKIP] Firefox not found on this machine." -ForegroundColor Gray
        Add-Summary "Firefox skipped: not installed on this machine." "INFO"
    } else {
        try {
            Write-Host "  Installed version: $($FirefoxFound[0].DisplayVersion)" -ForegroundColor Gray

            # Mozilla publishes the current release version as plain JSON - check it first so we
            # don't download ~55-60 MB for nothing if we're already up to date.
            $SkipFirefoxDownload = $false
            try {
                $VersionsJson = Invoke-RestMethod -Uri "https://product-details.mozilla.org/1.0/firefox_versions.json" -TimeoutSec 10 -ErrorAction Stop
                $LatestFxVersion = $VersionsJson.LATEST_FIREFOX_VERSION
                if ($LatestFxVersion) {
                    Write-Host "  Latest available version (per Mozilla): $LatestFxVersion" -ForegroundColor Gray
                    if ([version]$FirefoxFound[0].DisplayVersion -ge [version]$LatestFxVersion) {
                        $SkipFirefoxDownload = $true
                    }
                }
            } catch {
                Write-Host "  [INFO] Could not reach Mozilla's version API, will download and check directly." -ForegroundColor Gray
            }

            if ($SkipFirefoxDownload) {
                Write-Host "  [OK] Already up to date - nothing downloaded." -ForegroundColor Green
                Add-Summary "Firefox already up to date ($($FirefoxFound[0].DisplayVersion))." "INFO"
            } else {
                $FFFile = Join-Path $TempDownload "FirefoxSetup.exe"
                Write-Host "  -> Downloading latest Firefox from download.mozilla.org..." -ForegroundColor Yellow
                Invoke-WebRequest -Uri $FirefoxUrl -OutFile $FFFile -UseBasicParsing -ErrorAction Stop
                $SizeMB = [math]::Round((Get-Item $FFFile).Length / 1MB, 1)
                Write-Host "  [OK] Downloaded ($SizeMB MB)." -ForegroundColor Green

                Write-Host "  -> Installing silently (-ms) - this restarts Firefox if it's currently open..." -ForegroundColor Yellow
                $Process = Start-Process -FilePath $FFFile -ArgumentList "-ms" -Wait -NoNewWindow -PassThru -ErrorAction Stop

                if ($Process.ExitCode -eq 0) {
                    Write-Host "  [SUCCESS] Firefox updated." -ForegroundColor Green
                    Add-Summary "Firefox updated (was $($FirefoxFound[0].DisplayVersion))." "SUCCESS"
                } else {
                    Write-Host "  [WARNING] Installer returned exit code $($Process.ExitCode)." -ForegroundColor Yellow
                    Add-Summary "Firefox update returned exit code $($Process.ExitCode)." "ERROR"
                    Add-ErrorDetail -Component "Firefox update" -ExceptionMessage "Exit code $($Process.ExitCode)" -Context "File: $FFFile"
                }
            }
        } catch {
            Write-Host "  [ERROR] Failed to update Firefox: $_" -ForegroundColor Red
            Add-Summary "Error while updating Firefox: $_" "ERROR"
            Add-ErrorDetail -Component "Firefox" -ExceptionMessage $_.Exception.Message -Context "Url: $FirefoxUrl"
        } finally {
            if ($FFFile) { Remove-Item -Path $FFFile -Force -ErrorAction SilentlyContinue }
        }
    }

    # ------------------------------------------------------------------------------------------
    # POSTGRESQL / MYSQL SERVER / VEEAM BACKUP & REPLICATION
    # DETECTION + REPORTING ONLY - see explanation in the header comment block. Short version:
    # these three are infrastructure where a wrong silent unattended update could mean real
    # downtime or data-loss risk (DB engine in-place upgrade, backup product mid-job), so I
    # didn't want to wire up "just run the installer silently" without checking with you first
    # on which distribution/installer each of these actually uses on your VMs. This section
    # just tells you what's there and what Cyberwatch wants updated - nothing is touched.
    # ------------------------------------------------------------------------------------------
    Write-Host "`n[DATABASE / BACKUP SOFTWARE - DETECTION ONLY, NOT AUTO-UPDATED]" -ForegroundColor Blue

    $PgFound = Get-InstalledApps -NamePattern "PostgreSQL*"
    if ($PgFound) {
        foreach ($Pg in $PgFound) {
            Write-Host "  [FOUND] $($Pg.DisplayName) - version $($Pg.DisplayVersion) (update available per Cyberwatch, not applied)" -ForegroundColor Yellow
            Add-Summary "PostgreSQL detected: $($Pg.DisplayName) $($Pg.DisplayVersion) - update available, NOT applied (manual review)." "INFO"
        }
    } else {
        Write-Host "  [SKIP] PostgreSQL not found on this machine." -ForegroundColor Gray
    }

    $MySqlFound = Get-InstalledApps -NamePattern "MySQL Server*"
    if ($MySqlFound) {
        foreach ($My in $MySqlFound) {
            Write-Host "  [FOUND] $($My.DisplayName) - version $($My.DisplayVersion) (update available per Cyberwatch, not applied)" -ForegroundColor Yellow
            Add-Summary "MySQL Server detected: $($My.DisplayVersion) - update available, NOT applied (manual review)." "INFO"
        }
    } else {
        Write-Host "  [SKIP] MySQL Server not found on this machine." -ForegroundColor Gray
    }

    $VeeamFound = Get-InstalledApps -NamePattern "Veeam Backup*"
    if ($VeeamFound) {
        foreach ($Ve in $VeeamFound) {
            Write-Host "  [FOUND] $($Ve.DisplayName) - version $($Ve.DisplayVersion) (update available per Cyberwatch, not applied)" -ForegroundColor Yellow
            Add-Summary "Veeam Backup & Replication detected: $($Ve.DisplayVersion) - update available, NOT applied (manual review)." "INFO"
        }
    } else {
        Write-Host "  [SKIP] Veeam Backup & Replication not found on this machine." -ForegroundColor Gray
    }

} # end if ($InternetAvailable)

# ----------------------------------------------------------------------------------------------
# TEMPORARY FOLDER CLEANUP
# ----------------------------------------------------------------------------------------------
Write-Host "`n[CLEANUP] Removing temporary download folder..." -ForegroundColor Blue
if (Test-Path $TempDownload) {
    Remove-Item -Path $TempDownload -Recurse -Force -ErrorAction SilentlyContinue
    Write-Host "[OK] Temporary folder removed: $TempDownload" -ForegroundColor Green
}

# ----------------------------------------------------------------------------------------------
# SCHEDULED RESTART (20:00 tonight, never immediate - machine stays in prod during the day)
# ----------------------------------------------------------------------------------------------
if ($Global:RestartRequired) {
    Write-Host "`n[RESTART] One or more components need a reboot to finalize." -ForegroundColor Yellow

    # Cancel any previously pending shutdown from an earlier run, so re-running the script
    # today doesn't collide with a countdown already in progress.
    Start-Process -FilePath "shutdown.exe" -ArgumentList "/a" -Wait -NoNewWindow -ErrorAction SilentlyContinue | Out-Null

    $Now = Get-Date
    $TargetTime = Get-Date -Hour $ScheduledRestartHour -Minute $ScheduledRestartMinute -Second 0
    if ($Now -ge $TargetTime) { $TargetTime = $TargetTime.AddDays(1) }
    $DelaySeconds = [int]([math]::Ceiling(($TargetTime - $Now).TotalSeconds))

    Write-Host "  -> Scheduling restart for $($TargetTime.ToString('dd/MM/yyyy HH:mm')) ($DelaySeconds seconds from now)..." -ForegroundColor Yellow
    try {
        Start-Process -FilePath "shutdown.exe" -ArgumentList "/r", "/t", "$DelaySeconds", "/c", "Redemarrage planifie suite a mise a jour logicielle (Cyberwatch)" -Wait -NoNewWindow -PassThru -ErrorAction Stop | Out-Null
        Write-Host "  [OK] Restart scheduled." -ForegroundColor Green
        Add-Summary "Restart scheduled for $($TargetTime.ToString('dd/MM/yyyy HH:mm'))." "INFO"
    } catch {
        Write-Host "  [ERROR] Could not schedule the restart: $_" -ForegroundColor Red
        Add-Summary "Failed to schedule restart: $_" "ERROR"
        Add-ErrorDetail -Component "Restart scheduling" -ExceptionMessage $_.Exception.Message
    }
}

# ----------------------------------------------------------------------------------------------
# FINAL SUMMARY
# ----------------------------------------------------------------------------------------------
Write-Host "`n=======================================================================" -ForegroundColor Cyan
Write-Host " EXECUTION SUMMARY " -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan

if ($Global:SummaryLines.Count -eq 0) {
    Write-Host "Nothing to display." -ForegroundColor Gray
} else {
    foreach ($Line in $Global:SummaryLines) {
        if ($Line -match "^\[SUCCESS\]") { Write-Host $Line -ForegroundColor Green }
        elseif ($Line -match "^\[ERROR\]") { Write-Host $Line -ForegroundColor Red }
        else { Write-Host $Line -ForegroundColor Gray }
    }
}

Write-Host "-----------------------------------------------------------------------" -ForegroundColor Cyan
Write-Host " Total: $Global:SuccessCount success | $Global:ErrorCount error(s) | $Global:InfoCount info" -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan

if ($Global:ErrorCount -gt 0) {
    Write-Host "`n[WARNING] The script finished with at least one error. Review the summary above." -ForegroundColor Red
    Send-FailureNotification
} elseif ($Global:RestartRequired) {
    Add-ErrorDetail -Component "Post-update restart" -ExceptionMessage "A restart was scheduled to finalize one or more successful installations." -Context "See full summary below for which component(s) require it."
    Send-FailureNotification
    Write-Host "`n[OK] The script finished with no errors. Restart scheduled for tonight (see above)." -ForegroundColor Green
} else {
    Write-Host "`n[OK] The script finished with no errors." -ForegroundColor Green
}

Write-Host "`nScript finished." -ForegroundColor Cyan

try { Stop-Transcript | Out-Null } catch { }
Remove-Item -Path $LockFile -Force -ErrorAction SilentlyContinue

if ([Environment]::UserInteractive -and $Host.Name -match "ConsoleHost") {
    Write-Host "Press any key to close this window..." -ForegroundColor Cyan
    $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
}
```