````
# ==============================================================================================
# .NET Core and VC++ Redistributable auto-update script
# Single source: direct download from official Microsoft servers
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
$LockFile = "$env:TEMP\Update-DotNet-V2.lock"

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

# --- CONFIGURATION ---
$DotNetRoot   = "C:\Program Files\dotnet\shared"
$DotNetSdkRoot = "C:\Program Files\dotnet\sdk"
$RegistryVC   = "HKLM:\Software\Microsoft\VisualStudio\14.0\VC\Runtimes\x64"
$TempDownload = "$env:TEMP\DotNetUpdateTemp"
$VCRedistUrl  = "https://aka.ms/vs/17/release/vc_redist.x64.exe"   # Fixed URL, always redirects to the latest version

# --- EMAIL NOTIFICATION CONFIGURATION (used only if errors occur) ---
# Sent via Brevo's transactional email API (https://api.brevo.com)
$NotifyEmailTo   = "your-alert-address@example.com"
$NotifyEmailFrom = "your-sender-address@example.com"   # Must be a sender validated in your Brevo account
$BrevoApiKey     = "YOUR-BREVO-API-KEY-HERE"
$VMHostname      = $env:COMPUTERNAME

# Detect the machine's local IPv4 address(es) - excludes loopback and virtual/APIPA addresses
# so the email clearly identifies which physical/virtual machine on the network needs attention.
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
$Global:ErrorDetails    = @()   # Raw technical context per error, for direct copy-paste into an AI/debugging session
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

function Send-FailureNotification {
    # Sends a compact, technical email via Brevo's transactional API: exit codes and raw
    # context only, no decorative logs. Intended to be pasted directly into an AI assistant.
    if ($Global:ErrorDetails.Count -eq 0) {
        return
    }

    if ($Global:ErrorCount -gt 0) {
        $SubjectTag = "$($Global:ErrorCount) error(s)"
    } else {
        $SubjectTag = "restart pending"
    }
    $Subject = "[DotNet-Update] $SubjectTag on $VMHostname ($VMIPAddresses) - $(Get-Date -Format 'yyyy-MM-dd HH:mm')"

    $BodyLines = @()
    $BodyLines += "Automated .NET/VC++ update script failed."
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
        if ($Err.Context) {
            $BodyLines += "Context: $($Err.Context)"
        }
    }
    $BodyLines += ""
    $BodyLines += "--- FULL SUMMARY ---"
    $BodyLines += $Global:SummaryLines

    $PlainTextBody = $BodyLines -join "`n"
    # Brevo's API expects HTML content; wrapping in <pre> preserves line breaks and spacing as-is
    $HtmlBody = "<pre style='font-family:monospace; white-space:pre-wrap;'>$([System.Web.HttpUtility]::HtmlEncode($PlainTextBody))</pre>"

    $BrevoPayload = @{
        sender      = @{ email = $NotifyEmailFrom }
        to          = @(@{ email = $NotifyEmailTo })
        subject     = $Subject
        htmlContent = $HtmlBody
    } | ConvertTo-Json -Depth 5

    try {
        Add-Type -AssemblyName System.Web
        Invoke-RestMethod -Uri "https://api.brevo.com/v3/smtp/email" `
                           -Method Post `
                           -Headers @{ "api-key" = $BrevoApiKey; "Content-Type" = "application/json"; "Accept" = "application/json" } `
                           -Body $BrevoPayload `
                           -ErrorAction Stop | Out-Null
        Write-Host "[INFO] Failure notification email sent to $NotifyEmailTo via Brevo." -ForegroundColor Gray
    } catch {
        Write-Host "[WARNING] Could not send notification email via Brevo: $($_.Exception.Message)" -ForegroundColor Yellow
        Write-Host "          Check `$BrevoApiKey and `$NotifyEmailFrom (must be a validated sender in Brevo)." -ForegroundColor Yellow
    }
}

Write-Host "=======================================================================" -ForegroundColor Cyan
Write-Host " .NET Core / VC++ Redistributable check-and-update script " -ForegroundColor Cyan
Write-Host " Run date and time: $(Get-Date -Format 'dd/MM/yyyy HH:mm:ss')" -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan

Write-Host "`nThis script downloads the latest versions directly from official" -ForegroundColor DarkGray
Write-Host "Microsoft servers (aka.ms and raw.githubusercontent.com/dotnet/core)." -ForegroundColor DarkGray
Write-Host "A temporary folder will be created to store downloaded files, then" -ForegroundColor DarkGray
Write-Host "removed automatically at the end of the script.`n" -ForegroundColor DarkGray

# Create the temporary download folder
if (-not (Test-Path $TempDownload)) {
    New-Item -Path $TempDownload -ItemType Directory -Force | Out-Null
    Write-Host "[INFO] Temporary folder created: $TempDownload" -ForegroundColor Gray
}

# ----------------------------------------------------------------------------------------------
# PREREQUISITE CHECK: internet access
# ----------------------------------------------------------------------------------------------
Write-Host "`n[PREREQUISITE CHECK] Testing connection to Microsoft servers..." -ForegroundColor Blue

$InternetAvailable = $false
try {
    Invoke-WebRequest -Uri "https://aka.ms" -Method Head -TimeoutSec 8 -UseBasicParsing -ErrorAction Stop | Out-Null
    $InternetAvailable = $true
    Write-Host "[OK] Connection established. The script can query Microsoft normally." -ForegroundColor Green
} catch {
    Write-Host "[CRITICAL ERROR] Unable to reach Microsoft servers." -ForegroundColor Red
    Write-Host "                 Technical detail: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "                 Possible causes: no internet connection on this VM," -ForegroundColor Yellow
    Write-Host "                 a corporate proxy blocking outbound requests, or a firewall." -ForegroundColor Yellow
    Add-Summary "Failed to connect to Microsoft, script stopped." "ERROR"
    Add-ErrorDetail -Component "Prerequisite check (internet connectivity)" `
                    -ExceptionMessage $_.Exception.Message `
                    -Context "Invoke-WebRequest -Uri https://aka.ms -Method Head -TimeoutSec 8"

    Send-FailureNotification

    Write-Host "`n=======================================================================" -ForegroundColor Cyan
    Write-Host " SUMMARY " -ForegroundColor Cyan
    Write-Host "=======================================================================" -ForegroundColor Cyan
    $Global:SummaryLines | ForEach-Object { Write-Host $_ -ForegroundColor Red }
    Remove-Item -Path $LockFile -Force -ErrorAction SilentlyContinue
    if ([Environment]::UserInteractive -and $Host.Name -match "ConsoleHost") {
        Write-Host "`nPress any key to close this window..." -ForegroundColor Cyan
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit
}

# ----------------------------------------------------------------------------------------------
# PART 1: VISUAL C++ REDISTRIBUTABLE
# ----------------------------------------------------------------------------------------------
Write-Host "`n[STEP 1/2] Analyzing Visual C++ Redistributable..." -ForegroundColor Blue

$VCRedistLocalFile = Join-Path $TempDownload "VC_redist.x64.exe"

try {
    # STEP A - IDENTIFY THE CURRENTLY INSTALLED LOCAL VERSION (before any download)
    $InstalledVC     = Get-ItemProperty -Path $RegistryVC -ErrorAction SilentlyContinue
    $RawCurrentVC    = $InstalledVC.Version
    $CurrentVCVersion = $null

    if (-not $RawCurrentVC) {
        Write-Host "[INFO] No version of VC++ Redistributable is currently installed on this machine." -ForegroundColor Gray
    } else {
        # The registry sometimes stores this version with a leading "v" (e.g. "v14.44.35211.00"),
        # which breaks direct conversion to [version]. We extract only the numeric part, exactly
        # like we already do for the version read from the downloaded file below.
        if ($RawCurrentVC -match '(\d+(\.\d+){1,3})') {
            $CurrentVCVersion = $Matches[1]
            Write-Host "[INFO] Version currently installed on this VM: $CurrentVCVersion" -ForegroundColor Gray
        } else {
            Write-Host "[WARNING] Installed version format is unreadable ('$RawCurrentVC'). Treating as unknown; update will proceed." -ForegroundColor Yellow
        }
    }

    # STEP B - RETRIEVE THE REMOTE VERSION
    # Technical note: Microsoft does not publish the version number in this URL's HTTP headers.
    # The only reliable way to know it is to download the file and read its internal metadata.
    # This download is only used to IDENTIFY the remote version, not to install anything yet.
    Write-Host "[VC_REDIST] Downloading the file to identify the remote version (no installation at this stage)..." -ForegroundColor Yellow
    Write-Host "            URL used: $VCRedistUrl" -ForegroundColor DarkGray
    Invoke-WebRequest -Uri $VCRedistUrl -OutFile $VCRedistLocalFile -UseBasicParsing -ErrorAction Stop
    Write-Host "[OK] File retrieved." -ForegroundColor Green

    $RawVCVersion = [System.Diagnostics.FileVersionInfo]::GetVersionInfo($VCRedistLocalFile).FileVersion

    # The version number returned by Microsoft sometimes has a text prefix (e.g. "v14.4.35211.00")
    # which prevents direct conversion to [version]. We extract only the numeric part.
    if ($RawVCVersion -match '(\d+(\.\d+){1,3})') {
        $SourceVCVersion = $Matches[1]
        Write-Host "[INFO] Version available from Microsoft: $SourceVCVersion" -ForegroundColor Gray
    } else {
        throw "The version format returned by the file is unreadable: '$RawVCVersion'"
    }

    # STEP C - STRICT COMPARISON: nothing happens past this point if the version is not newer
    if (-not $CurrentVCVersion -or ([version]$SourceVCVersion -gt [version]$CurrentVCVersion)) {

        # STEP D - INSTALLATION (only reached if the comparison above is positive)
        Write-Host "[VC_REDIST] Newer version confirmed. Running silent installation..." -ForegroundColor Yellow
        Write-Host "            (This can take 30 seconds to a few minutes depending on the VM.)" -ForegroundColor DarkGray

        $VCArguments = "/install /quiet /norestart"
        $VCProcess   = Start-Process -FilePath $VCRedistLocalFile -ArgumentList $VCArguments -Wait -NoNewWindow -PassThru

        # STEP E - VERIFY SUCCESS BEFORE ANY OTHER ACTION
        if ($VCProcess.ExitCode -eq 0 -or $VCProcess.ExitCode -eq 3010) {
            Write-Host "[SUCCESS] VC++ Redistributable updated to version $SourceVCVersion." -ForegroundColor Green
            Add-Summary "VC++ Redistributable installed successfully (version $SourceVCVersion)." "SUCCESS"
            if ($VCProcess.ExitCode -eq 3010) {
                Write-Host "          Exit code 3010: a VM restart is required to finalize the installation." -ForegroundColor Yellow
                Add-Summary "Restart required to finalize VC++ Redistributable." "INFO"
                $Global:RestartRequired = $true
            }
            # IMPORTANT NOTE: unlike .NET Core, VC++ Redistributable does not keep multiple
            # versions side by side on disk - Microsoft overwrites the existing installation in place.
            # There is therefore no old version folder to clean up here, by Microsoft's design.
        } else {
            Write-Host "[ERROR] The installer returned an unusual exit code: $($VCProcess.ExitCode)" -ForegroundColor Red
            Write-Host "        This usually indicates an installation failure. Check the Windows logs" -ForegroundColor Red
            Write-Host "        (Event Viewer > Applications) for more detail." -ForegroundColor Red
            Add-Summary "VC++ Redistributable installation failed (code $($VCProcess.ExitCode))." "ERROR"
            Add-ErrorDetail -Component "VC++ Redistributable installation" `
                            -ExceptionMessage "Installer exit code: $($VCProcess.ExitCode)" `
                            -Context "Start-Process -FilePath $VCRedistLocalFile -ArgumentList '/install /quiet /norestart'; source version $SourceVCVersion, local version $CurrentVCVersion"
        }
    } else {
        Write-Host "[INFO] VC++ Redistributable is already up to date (version $CurrentVCVersion). No installation launched." -ForegroundColor Gray
        Add-Summary "VC++ Redistributable already up to date (version $CurrentVCVersion)." "INFO"

        # The file downloaded for comparison purposes is now useless: delete it immediately
        # instead of waiting for the final cleanup, so nothing unnecessary is left on disk.
        Remove-Item -Path $VCRedistLocalFile -Force -ErrorAction SilentlyContinue
        Write-Host "[INFO] Temporary comparison file deleted (not used)." -ForegroundColor Gray
    }
} catch {
    Write-Host "[ERROR] A problem occurred while processing VC++ Redistributable." -ForegroundColor Red
    Write-Host "        Technical detail: $_" -ForegroundColor Red
    Add-Summary "Error while processing VC++ Redistributable: $_" "ERROR"
    Add-ErrorDetail -Component "VC++ Redistributable (general)" `
                    -ExceptionMessage $_.Exception.Message `
                    -Context "URL: $VCRedistUrl ; local file: $VCRedistLocalFile"
}

# ----------------------------------------------------------------------------------------------
# PART 2: .NET CORE / ASP.NET CORE / WINDOWS DESKTOP RUNTIME
# ----------------------------------------------------------------------------------------------
Write-Host "`n[STEP 2/3] Analyzing locally installed .NET Core runtimes..." -ForegroundColor Blue

if (-not (Test-Path $DotNetRoot)) {
    Write-Host "[INFO] The folder '$DotNetRoot' does not exist on this machine." -ForegroundColor Gray
    Write-Host "       This means no .NET Core runtime is installed here." -ForegroundColor Gray
    Write-Host "       For safety, the script will not install anything at this step." -ForegroundColor Gray
    Add-Summary ".NET Core not installed on this machine, step skipped." "INFO"
} else {

    # Mapping table: local folder name <=> file prefix <=> Microsoft GitHub API key
    $RuntimeMap = @{
        "Microsoft.NETCore.App"        = @{ Prefix = "dotnet-runtime";        ApiKey = "runtime" }
        "Microsoft.AspNetCore.App"     = @{ Prefix = "aspnetcore-runtime";     ApiKey = "aspnetcore-runtime" }
        "Microsoft.WindowsDesktop.App" = @{ Prefix = "windowsdesktop-runtime"; ApiKey = "windowsdesktop" }
    }

    $LocalRuntimeFolders = Get-ChildItem -Path $DotNetRoot -Directory -ErrorAction SilentlyContinue

    if (-not $LocalRuntimeFolders) {
        Write-Host "[INFO] The folder $DotNetRoot exists but contains no recognized runtime." -ForegroundColor Gray
    }

    foreach ($RuntimeFolder in $LocalRuntimeFolders) {
        $RuntimeName = $RuntimeFolder.Name

        if (-not $RuntimeMap.ContainsKey($RuntimeName)) {
            Write-Host "`n[INFO] Folder '$RuntimeName' detected but not handled by this script (third-party or unknown component). Skipped." -ForegroundColor Gray
            continue
        }

        $FilePrefix = $RuntimeMap[$RuntimeName].Prefix
        $ApiKey     = $RuntimeMap[$RuntimeName].ApiKey

        Write-Host "`n--> Analyzing component: $RuntimeName" -ForegroundColor Cyan

        $LocalVersions = Get-ChildItem -Path $RuntimeFolder.FullName -Directory |
                         Where-Object { $_.Name -match "^\d+\." } |
                         ForEach-Object { [version]$_.Name }

        if (-not $LocalVersions) {
            Write-Host "    [INFO] No valid version subfolder found in $RuntimeName. Moving on." -ForegroundColor Gray
            continue
        }

        $MajorVersions = $LocalVersions | ForEach-Object { $_.Major } | Select-Object -Unique

        foreach ($Major in $MajorVersions) {
            $MaxLocalVersion = ($LocalVersions | Where-Object { $_.Major -eq $Major } | Sort-Object)[-1]
            Write-Host "    Major version $Major detected. Latest local version: $MaxLocalVersion" -ForegroundColor Gray

            try {
                # STEP A - QUERY THE OFFICIAL METADATA (lightweight JSON, no big file download yet)
                $ReleasesUrl = "https://raw.githubusercontent.com/dotnet/core/main/release-notes/$Major.0/releases.json"
                Write-Host "    [LOOKUP] Querying the official Microsoft source for major version $Major..." -ForegroundColor Yellow
                Write-Host "             URL: $ReleasesUrl" -ForegroundColor DarkGray

                $ReleasesData  = Invoke-RestMethod -Uri $ReleasesUrl -TimeoutSec 15 -ErrorAction Stop
                $LatestRelease = $ReleasesData.releases | Select-Object -First 1
                $ComponentBlock = $LatestRelease.$ApiKey

                if (-not $ComponentBlock) {
                    Write-Host "    [INFO] Microsoft has not published information for this component in major version $Major." -ForegroundColor Gray
                    Add-Summary "$RuntimeName (major $Major): no data available from Microsoft." "INFO"
                    continue
                }

                $SourceVersion = [version]$ComponentBlock.version
                Write-Host "    [INFO] Latest version published by Microsoft: $SourceVersion" -ForegroundColor Gray

                # STEP B - STRICT COMPARISON: if not newer, stop here, NO download happens
                if ($SourceVersion -le $MaxLocalVersion) {
                    Write-Host "    [INFO] Component already up to date (Local: $MaxLocalVersion / Microsoft: $SourceVersion). No action." -ForegroundColor Gray
                    Add-Summary "$RuntimeName already up to date (version $MaxLocalVersion)." "INFO"
                    continue
                }

                $Win64File = $ComponentBlock.files | Where-Object { $_.name -match "win-x64.*\.exe$" } | Select-Object -First 1

                if (-not $Win64File) {
                    Write-Host "    [ERROR] A newer version exists ($SourceVersion) but no win-x64 executable" -ForegroundColor Red
                    Write-Host "            is provided by Microsoft for this component. Installation not possible." -ForegroundColor Red
                    Add-Summary "${RuntimeName}: newer version $SourceVersion found but no win-x64 installer." "ERROR"
                    Add-ErrorDetail -Component "$RuntimeName (major $Major)" `
                                    -ExceptionMessage "No win-x64 .exe file found in ComponentBlock.files for version $SourceVersion" `
                                    -Context "ApiKey used: $ApiKey ; ReleasesUrl: $ReleasesUrl"
                    continue
                }

                # STEP C - DOWNLOAD (only reached because the comparison above was positive)
                Write-Host "    [MATCH] Newer version available: $SourceVersion (current local: $MaxLocalVersion)" -ForegroundColor Green
                Write-Host "    [DOWNLOAD] Fetching the installer from Microsoft..." -ForegroundColor Yellow

                $DownloadedFile = Join-Path $TempDownload "$FilePrefix-$SourceVersion-win-x64.exe"
                Invoke-WebRequest -Uri $Win64File.url -OutFile $DownloadedFile -UseBasicParsing -ErrorAction Stop
                Write-Host "    [OK] Download complete: $DownloadedFile" -ForegroundColor Green

                # STEP D - INSTALLATION
                Write-Host "    -> Running silent installation for $RuntimeName..." -ForegroundColor Yellow
                $Arguments = "/quiet /norestart"
                $Process   = Start-Process -FilePath $DownloadedFile -ArgumentList $Arguments -Wait -NoNewWindow -PassThru -ErrorAction Stop

                # STEP E - CLEANUP OF OLD VERSIONS, ONLY IF INSTALLATION SUCCEEDED
                if ($Process.ExitCode -eq 0 -or $Process.ExitCode -eq 3010) {
                    Write-Host "    [SUCCESS] Installation of version $SourceVersion completed successfully." -ForegroundColor Green
                    Add-Summary "$RuntimeName updated to $SourceVersion." "SUCCESS"
                    if ($Process.ExitCode -eq 3010) {
                        Write-Host "    Note: VM restart required to finalize." -ForegroundColor Yellow
                        Add-Summary "Restart required to finalize $RuntimeName." "INFO"
                        $Global:RestartRequired = $true
                    }

                    Start-Sleep -Seconds 5
                    Write-Host "    -> Cleaning up old versions for major $Major..." -ForegroundColor Yellow
                    $CurrentFolders = Get-ChildItem -Path $RuntimeFolder.FullName -Directory
                    foreach ($Folder in $CurrentFolders) {
                        if ($Folder.Name -match "^\d+\.") {
                            $FolderVer = [version]$Folder.Name
                            if ($FolderVer.Major -eq $Major -and $FolderVer -lt $SourceVersion) {
                                Write-Host "       [CLEANUP] Removing obsolete folder: $($Folder.Name)" -ForegroundColor Magenta
                                Remove-Item -Path $Folder.FullName -Recurse -Force -ErrorAction SilentlyContinue
                            }
                        }
                    }
                } else {
                    Write-Host "    [ERROR] Unusual exit code: $($Process.ExitCode). No cleanup performed, for safety." -ForegroundColor Red
                    Add-Summary "$RuntimeName installation failed (code $($Process.ExitCode))." "ERROR"
                    Add-ErrorDetail -Component "$RuntimeName (major $Major) installation" `
                                    -ExceptionMessage "Installer exit code: $($Process.ExitCode)" `
                                    -Context "Start-Process -FilePath $DownloadedFile -ArgumentList '/quiet /norestart'; target version $SourceVersion, previous local version $MaxLocalVersion"
                }

            } catch {
                Write-Host "    [ERROR] Unable to check or install $RuntimeName (major $Major)." -ForegroundColor Red
                Write-Host "            Technical detail: $_" -ForegroundColor Red
                Write-Host "            Possible causes: Microsoft has not yet published this major version on GitHub," -ForegroundColor Yellow
                Write-Host "            a temporary network error, or a format change on Microsoft's side." -ForegroundColor Yellow
                Add-Summary "Error while processing $RuntimeName (major $Major): $_" "ERROR"
                Add-ErrorDetail -Component "$RuntimeName (major $Major, general)" `
                                -ExceptionMessage $_.Exception.Message `
                                -Context "ReleasesUrl: https://raw.githubusercontent.com/dotnet/core/main/release-notes/$Major.0/releases.json ; ApiKey: $ApiKey"
            }
        }
    }
}

# ----------------------------------------------------------------------------------------------
# PART 3: .NET SDK
# ----------------------------------------------------------------------------------------------
Write-Host "`n[STEP 3/3] Analyzing locally installed .NET SDK..." -ForegroundColor Blue

if (-not (Test-Path $DotNetSdkRoot)) {
    Write-Host "[INFO] The folder '$DotNetSdkRoot' does not exist on this machine." -ForegroundColor Gray
    Write-Host "       This means no .NET SDK is installed here (this is normal on a pure hosting/production VM)." -ForegroundColor Gray
    Add-Summary ".NET SDK not installed on this machine, step skipped." "INFO"
} else {

    # SDK folders are named directly with their version (e.g. "8.0.404", "9.0.314"), no sub-component split
    $LocalSdkVersions = Get-ChildItem -Path $DotNetSdkRoot -Directory -ErrorAction SilentlyContinue |
                        Where-Object { $_.Name -match "^\d+\." } |
                        ForEach-Object { [version]$_.Name }

    if (-not $LocalSdkVersions) {
        Write-Host "[INFO] No valid SDK version subfolder found in $DotNetSdkRoot." -ForegroundColor Gray
    } else {

        $SdkMajorVersions = $LocalSdkVersions | ForEach-Object { $_.Major } | Select-Object -Unique

        foreach ($Major in $SdkMajorVersions) {
            $MaxLocalSdkVersion = ($LocalSdkVersions | Where-Object { $_.Major -eq $Major } | Sort-Object)[-1]
            Write-Host "`n--> Analyzing SDK major version $Major. Latest local version: $MaxLocalSdkVersion" -ForegroundColor Cyan

            try {
                $ReleasesUrl = "https://raw.githubusercontent.com/dotnet/core/main/release-notes/$Major.0/releases.json"
                Write-Host "    [LOOKUP] Querying the official Microsoft source for SDK major $Major..." -ForegroundColor Yellow
                Write-Host "             URL: $ReleasesUrl" -ForegroundColor DarkGray

                $ReleasesData  = Invoke-RestMethod -Uri $ReleasesUrl -TimeoutSec 15 -ErrorAction Stop
                $LatestRelease = $ReleasesData.releases | Select-Object -First 1
                $SdkBlock      = $LatestRelease.sdk

                if (-not $SdkBlock) {
                    Write-Host "    [INFO] Microsoft has not published SDK information for major $Major." -ForegroundColor Gray
                    Add-Summary "SDK (major $Major): no data available from Microsoft." "INFO"
                    continue
                }

                $SourceSdkVersion = [version]$SdkBlock.version
                Write-Host "    [INFO] Latest SDK version published by Microsoft: $SourceSdkVersion" -ForegroundColor Gray

                # STRICT COMPARISON: if not newer, stop here, NO download happens
                if ($SourceSdkVersion -le $MaxLocalSdkVersion) {
                    Write-Host "    [INFO] SDK already up to date (Local: $MaxLocalSdkVersion / Microsoft: $SourceSdkVersion). No action." -ForegroundColor Gray
                    Add-Summary "SDK already up to date (version $MaxLocalSdkVersion)." "INFO"
                    continue
                }

                $Win64File = $SdkBlock.files | Where-Object { $_.name -match "win-x64\.exe$" } | Select-Object -First 1

                if (-not $Win64File) {
                    Write-Host "    [ERROR] A newer SDK version exists ($SourceSdkVersion) but no win-x64 executable" -ForegroundColor Red
                    Write-Host "            is provided by Microsoft. Installation not possible." -ForegroundColor Red
                    Add-Summary "SDK: newer version $SourceSdkVersion found but no win-x64 installer." "ERROR"
                    Add-ErrorDetail -Component "SDK (major $Major)" `
                                    -ExceptionMessage "No win-x64 .exe file found in SdkBlock.files for version $SourceSdkVersion" `
                                    -Context "ReleasesUrl: $ReleasesUrl"
                    continue
                }

                # DOWNLOAD (only reached because the comparison above was positive)
                Write-Host "    [MATCH] Newer SDK version available: $SourceSdkVersion (current local: $MaxLocalSdkVersion)" -ForegroundColor Green
                Write-Host "    [DOWNLOAD] Fetching the SDK installer from Microsoft..." -ForegroundColor Yellow

                $DownloadedFile = Join-Path $TempDownload "dotnet-sdk-$SourceSdkVersion-win-x64.exe"
                Invoke-WebRequest -Uri $Win64File.url -OutFile $DownloadedFile -UseBasicParsing -ErrorAction Stop
                Write-Host "    [OK] Download complete: $DownloadedFile" -ForegroundColor Green

                # INSTALLATION
                Write-Host "    -> Running silent installation for .NET SDK..." -ForegroundColor Yellow
                $Arguments = "/quiet /norestart"
                $Process   = Start-Process -FilePath $DownloadedFile -ArgumentList $Arguments -Wait -NoNewWindow -PassThru -ErrorAction Stop

                # CLEANUP OF OLD VERSIONS, ONLY IF INSTALLATION SUCCEEDED
                if ($Process.ExitCode -eq 0 -or $Process.ExitCode -eq 3010) {
                    Write-Host "    [SUCCESS] SDK installation of version $SourceSdkVersion completed successfully." -ForegroundColor Green
                    Add-Summary "SDK updated to $SourceSdkVersion." "SUCCESS"
                    if ($Process.ExitCode -eq 3010) {
                        Write-Host "    Note: VM restart required to finalize." -ForegroundColor Yellow
                        Add-Summary "Restart required to finalize SDK update." "INFO"
                        $Global:RestartRequired = $true
                    }

                    Start-Sleep -Seconds 5
                    Write-Host "    -> Cleaning up old SDK versions for major $Major..." -ForegroundColor Yellow
                    $CurrentSdkFolders = Get-ChildItem -Path $DotNetSdkRoot -Directory
                    foreach ($Folder in $CurrentSdkFolders) {
                        if ($Folder.Name -match "^\d+\.") {
                            $FolderVer = [version]$Folder.Name
                            if ($FolderVer.Major -eq $Major -and $FolderVer -lt $SourceSdkVersion) {
                                Write-Host "       [CLEANUP] Removing obsolete SDK folder: $($Folder.Name)" -ForegroundColor Magenta
                                Remove-Item -Path $Folder.FullName -Recurse -Force -ErrorAction SilentlyContinue
                            }
                        }
                    }
                } else {
                    Write-Host "    [ERROR] Unusual exit code: $($Process.ExitCode). No cleanup performed, for safety." -ForegroundColor Red
                    Add-Summary "SDK installation failed (code $($Process.ExitCode))." "ERROR"
                    Add-ErrorDetail -Component "SDK (major $Major) installation" `
                                    -ExceptionMessage "Installer exit code: $($Process.ExitCode)" `
                                    -Context "Start-Process -FilePath $DownloadedFile -ArgumentList '/quiet /norestart'; target version $SourceSdkVersion, previous local version $MaxLocalSdkVersion"
                }

            } catch {
                Write-Host "    [ERROR] Unable to check or install .NET SDK (major $Major)." -ForegroundColor Red
                Write-Host "            Technical detail: $_" -ForegroundColor Red
                Add-Summary "Error while processing SDK (major $Major): $_" "ERROR"
                Add-ErrorDetail -Component "SDK (major $Major, general)" `
                                -ExceptionMessage $_.Exception.Message `
                                -Context "ReleasesUrl: https://raw.githubusercontent.com/dotnet/core/main/release-notes/$Major.0/releases.json"
            }
        }
    }
}

# ----------------------------------------------------------------------------------------------
# TEMPORARY FOLDER CLEANUP
# ----------------------------------------------------------------------------------------------
Write-Host "`n[CLEANUP] Removing temporary download folder..." -ForegroundColor Blue
if (Test-Path $TempDownload) {
    Remove-Item -Path $TempDownload -Recurse -Force -ErrorAction SilentlyContinue
    Write-Host "[OK] Temporary folder removed: $TempDownload" -ForegroundColor Green
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
        if ($Line -match "^\[SUCCESS\]") {
            Write-Host $Line -ForegroundColor Green
        } elseif ($Line -match "^\[ERROR\]") {
            Write-Host $Line -ForegroundColor Red
        } else {
            Write-Host $Line -ForegroundColor Gray
        }
    }
}

Write-Host "-----------------------------------------------------------------------" -ForegroundColor Cyan
Write-Host " Total: $Global:SuccessCount success | $Global:ErrorCount error(s) | $Global:InfoCount info" -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan

if ($Global:RestartRequired) {
    Write-Host "`n[RESTART REQUIRED] One or more components were updated but need a VM restart" -ForegroundColor Yellow
    Write-Host "                   to fully take effect. Plan a reboot at your convenience." -ForegroundColor Yellow
}

if ($Global:ErrorCount -gt 0) {
    Write-Host "`n[WARNING] The script finished with at least one error. Review the summary above" -ForegroundColor Red
    Write-Host "          and the matching detail earlier in the log to understand the cause." -ForegroundColor Red
    Send-FailureNotification
} elseif ($Global:RestartRequired) {
    # No errors, but flag the pending restart via email too - easy to miss otherwise across many VMs
    Add-ErrorDetail -Component "Post-update restart" `
                    -ExceptionMessage "A VM restart is pending to finalize one or more successful installations." `
                    -Context "See full summary below for which component(s) require it."
    Send-FailureNotification
    Write-Host "`n[OK] The script finished with no errors, but a restart is pending (see above)." -ForegroundColor Green
} else {
    Write-Host "`n[OK] The script finished with no errors." -ForegroundColor Green
}

Write-Host "`nScript finished." -ForegroundColor Cyan

# Release the anti-overlap lock now that the script has finished normally
Remove-Item -Path $LockFile -Force -ErrorAction SilentlyContinue

# Only wait for a keypress when running interactively (double-click or manual console launch).
# In a scheduled task, $Host.Name is usually "Default Host" or non-interactive, so this is skipped
# automatically - no need to maintain two separate script versions.
if ([Environment]::UserInteractive -and $Host.Name -match "ConsoleHost") {
    Write-Host "Press any key to close this window..." -ForegroundColor Cyan
    $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
}
```