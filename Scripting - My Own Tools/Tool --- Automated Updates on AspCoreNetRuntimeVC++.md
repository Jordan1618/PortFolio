# ==============================================================================================
# .NET Core and VC++ Redistributable auto-update script
# Single source: direct download from official Microsoft servers
# ==============================================================================================

# --- CONFIGURATION ---
$DotNetRoot   = "C:\Program Files\dotnet\shared"
$RegistryVC   = "HKLM:\Software\Microsoft\VisualStudio\14.0\VC\Runtimes\x64"
$TempDownload = "$env:TEMP\DotNetUpdateTemp"
$VCRedistUrl  = "https://aka.ms/vs/17/release/vc_redist.x64.exe"   # Fixed URL, always redirects to the latest version

# Counters and lines for the final summary
$Global:SuccessCount = 0
$Global:ErrorCount   = 0
$Global:InfoCount    = 0
$Global:SummaryLines = @()

function Add-Summary {
    param([string]$Message, [string]$Type = "INFO")
    $Global:SummaryLines += "[$Type] $Message"
    switch ($Type) {
        "SUCCESS" { $Global:SuccessCount++ }
        "ERROR"   { $Global:ErrorCount++ }
        default   { $Global:InfoCount++ }
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

    Write-Host "`n=======================================================================" -ForegroundColor Cyan
    Write-Host " SUMMARY " -ForegroundColor Cyan
    Write-Host "=======================================================================" -ForegroundColor Cyan
    $Global:SummaryLines | ForEach-Object { Write-Host $_ -ForegroundColor Red }
    Write-Host "`nPress any key to close this window..." -ForegroundColor Cyan
    $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
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
            }
            # IMPORTANT NOTE: unlike .NET Core, VC++ Redistributable does not keep multiple
            # versions side by side on disk - Microsoft overwrites the existing installation in place.
            # There is therefore no old version folder to clean up here, by Microsoft's design.
        } else {
            Write-Host "[ERROR] The installer returned an unusual exit code: $($VCProcess.ExitCode)" -ForegroundColor Red
            Write-Host "        This usually indicates an installation failure. Check the Windows logs" -ForegroundColor Red
            Write-Host "        (Event Viewer > Applications) for more detail." -ForegroundColor Red
            Add-Summary "VC++ Redistributable installation failed (code $($VCProcess.ExitCode))." "ERROR"
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
}

# ----------------------------------------------------------------------------------------------
# PART 2: .NET CORE / ASP.NET CORE / WINDOWS DESKTOP RUNTIME
# ----------------------------------------------------------------------------------------------
Write-Host "`n[STEP 2/2] Analyzing locally installed .NET Core runtimes..." -ForegroundColor Blue

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
                }

            } catch {
                Write-Host "    [ERROR] Unable to check or install $RuntimeName (major $Major)." -ForegroundColor Red
                Write-Host "            Technical detail: $_" -ForegroundColor Red
                Write-Host "            Possible causes: Microsoft has not yet published this major version on GitHub," -ForegroundColor Yellow
                Write-Host "            a temporary network error, or a format change on Microsoft's side." -ForegroundColor Yellow
                Add-Summary "Error while processing $RuntimeName (major $Major): $_" "ERROR"
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

if ($Global:ErrorCount -gt 0) {
    Write-Host "`n[WARNING] The script finished with at least one error. Review the summary above" -ForegroundColor Red
    Write-Host "          and the matching detail earlier in the log to understand the cause." -ForegroundColor Red
} else {
    Write-Host "`n[OK] The script finished with no errors." -ForegroundColor Green
}

Write-Host "`nPress any key to close this window..." -ForegroundColor Cyan
$null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
