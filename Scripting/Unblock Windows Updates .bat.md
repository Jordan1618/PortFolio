### **To unblock the Windows Update Service and User Restrictions :** 
- I needed to unblock for all users the ability to update their computers, schedulling updates by default at 8pm and includes facultative updates. Yes i use AI but i understand each lign and its my biggest success and it helps me to understand the regedit structure.

---

**Script Part :**

@echo off
:: UTF-8 encoding to correctly handle accents and special characters
chcp 65001 >nul
cls

echo =================================================================
echo   FULL UNBLOCKING AND ADVANCED CONFIGURATION OF WINDOWS UPDATE
echo =================================================================
echo.
echo [INFO] Validating administrator privileges... [cite: 5, 16]

:: Checking for administrator privileges
net session >nul 2>&1
if %errorLevel% == 0 (
    echo [SUCCESS] The script is running with the required privileges.
) else (
    echo [ERROR] CRITICAL: You must run this script as an administrator.
    echo         Right-click on the file -> "Run as administrator".
    goto END_ERROR
)

echo.
echo -----------------------------------------------------------------
echo [PROGRESS 1/4] Elevating Windows Update privileges for non-admins
echo -----------------------------------------------------------------
:: Formally allows non-administrators to receive notifications and install updates
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ElevateNonAdmins" /t REG_DWORD /d 1 /f >nul
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ElevateNonAdmins" /t REG_DWORD /d 1 /f >nul

if %errorLevel% == 0 (
    echo [OK] Installation rights granted to standard users. [cite: 17]
) else (
    echo [ERROR] Failed to assign privileges. [cite: 17]
    goto END_ERROR
)

echo.
echo -----------------------------------------------------------------
echo [PROGRESS 2/4] Removing access blocks and restrictions
echo -----------------------------------------------------------------
:: Disables the policy that could prohibit access to Windows Update features [cite: 18]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "NoWindowsUpdate" /t REG_DWORD /d 0 /f >nul

:: Ensures users have access to the Windows Update GUI in Settings [cite: 18]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "SettingsPageVisibility" /t REG_SZ /d "show:windowsupdate" /f >nul

if %errorLevel% == 0 (
    echo [OK] Interface and feature blocks have been lifted. [cite: 18]
) else (
    echo [ERROR] Failed to remove blocks. [cite: 18]
    goto END_ERROR
)

echo.
echo -----------------------------------------------------------------
echo [PROGRESS 3/4] Configuring daily scheduling at 8:00 PM (20:00)
echo -----------------------------------------------------------------
:: AUOptions 4 = Automatic download and schedule [cite: 7]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "AUOptions" /t REG_DWORD /d 4 /f >nul
:: ScheduledInstallDay 0 = Every day [cite: 7]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ScheduledInstallDay" /t REG_DWORD /d 0 /f >nul
:: ScheduledInstallTime 20 = 8:00 PM (24h format) [cite: 7]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ScheduledInstallTime" /t REG_DWORD /d 20 /f >nul

:: Force local policies to enforce custom scheduling [cite: 7]
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "NoAutoUpdate" /t REG_DWORD /d 0 /f >nul
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "AUOptions" /t REG_DWORD /d 4 /f >nul
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ScheduledInstallDay" /t REG_DWORD /d 0 /f >nul
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ScheduledInstallTime" /t REG_DWORD /d 20 /f >nul [cite: 8]

if %errorLevel% == 0 (
    echo [OK] Scheduling successfully configured for every day at 20:00.
) else (
    echo [ERROR] Failed to configure update scheduling. [cite: 9]
    goto END_ERROR
)

echo.
echo -----------------------------------------------------------------
echo [PROGRESS 4/4] Including Drivers and optional updates
echo -----------------------------------------------------------------
:: Include driver updates via Windows Update [cite: 10]
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /v "ExcludeWUDriversInQualityUpdate" /t REG_DWORD /d 0 /f >nul
:: Include other Microsoft products (e.g., Office) and optional content [cite: 10]
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "AllowMUUpdateService" /t REG_DWORD /d 1 /f >nul

if %errorLevel% == 0 (
    echo [OK] Driver and optional update inclusion is now enabled.
) else (
    echo [ERROR] Failed to enable drivers/optional updates inclusion.
    goto END_ERROR
)

echo.
echo -----------------------------------------------------------------
echo [PROGRESS EXECUTING] Applying and restarting services
echo -----------------------------------------------------------------
echo [INFO] Refreshing local group policies... [cite: 19]
gpupdate /force >nul 2>&1

echo [INFO] Restarting Windows Update service... [cite: 11, 19]
net stop wuauserv >nul 2>&1
net start wuauserv >nul 2>&1

if %errorLevel% == 0 (
    echo [OK] Services and policies successfully refreshed.
) else (
    echo [WARNING] Minor error encountered during service restart. A PC reboot is recommended. [cite: 11]
)

echo.
echo =================================================================
echo   CONFIGURATION AND UNBLOCKING COMPLETED!
echo =================================================================
echo Standard users can now check for updates and launch 
echo installations without any UAC (admin) prompt. [cite: 20]
echo.
echo Note: Windows can take several hours to align its internal [cite: 13]
echo scheduled tasks with these registry modifications. [cite: 13]
goto END [cite: 13, 21]

:END_ERROR
echo.
echo =================================================================
echo   SCRIPT FAILED [cite: 14]
echo =================================================================
echo Please verify the error messages above and ensure you ran this file 
echo by right-clicking on it -> "Run as administrator".

:END [cite: 22]
echo.
pause