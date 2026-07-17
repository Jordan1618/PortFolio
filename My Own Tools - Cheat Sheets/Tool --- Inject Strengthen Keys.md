```batch
@echo off
title Configuration Registre & Force Upgrade Windows 11 25H2
chcp 65001 >nul

:: Vérification et demande automatique des droits Administrateur
fltmc >nul 2>&1 || (
  echo [INFO] Activation des droits administrateur requis...
  powershell -nop -c "start cmd -args '/d/x/r_\"\"%~f0\"\" %*' -verb runas" && exit /b
)

cls
echo ===================================================================
echo   BYPASS TOTAL MATÉRIEL - PASSAGE FORCE VERS WINDOWS 11 25H2
echo ===================================================================
echo.

echo [1/3] Injection des clés MoSetup et LabConfig...

:: Méthode 1 : MoSetup
set "target_mosetup=HKLM\SYSTEM\Setup\MoSetup"
reg add "%target_mosetup%" /v "AllowUpgradesWithUnsupportedTPMOrCPU" /t REG_DWORD /d 1 /f >nul
reg add "%target_mosetup%" /v "SkipTPMCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_mosetup%" /v "SkipCPUCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_mosetup%" /v "SkipSecureBootCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_mosetup%" /v "SkipRAMCheck" /t REG_DWORD /d 1 /f >nul

:: Méthode 2 : LabConfig (Bypass profond de l'installeur)
set "target_labconfig=HKLM\SYSTEM\Setup\LabConfig"
reg add "%target_labconfig%" /v "BypassTPMCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_labconfig%" /v "BypassCPUCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_labconfig%" /v "BypassSecureBootCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_labconfig%" /v "BypassRAMCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_labconfig%" /v "BypassStorageCheck" /t REG_DWORD /d 1 /f >nul

echo [2/3] Registre configuré avec succès !
echo.
echo [3/3] Recherche de l'ISO Windows 11 25H2 montée...

:: Recherche automatique du lecteur virtuel où l'ISO est montée
set "drive_letter="
for %%d in (D E F G H I J K L M N O P Q R S T U V W X Y Z) do (
    if exist "%%d:\sources\setupprep.exe" (
        set "drive_letter=%%d"
        goto :found
    )
    if exist "%%d:\setup.exe" (
        set "drive_letter=%%d"
        goto :found
    )
)

:found
if "%drive_letter%"=="" (
    echo [ERREUR] Impossible de trouver l'ISO Windows 11 installée.
    echo Assurez-vous d'avoir double-cliqué sur votre fichier ISO pour le "monter" avant de lancer ce script.
    echo.
    pause
    exit /b
)

echo.
echo ===================================================================
echo  ISO DEPLOYÉE TROUVÉE SUR LE LECTEUR [%drive_letter%:]
echo ===================================================================
echo  Le script va lancer l'installation en mode Bypass de la 25H2.
echo  Pensez à couper votre connexion Internet (Wi-Fi / Câble) maintenant.
echo ===================================================================
echo.
pause

echo Lancement du processus d'installation...
:: Exécution via l'argument /product server pour esquiver le contrôle d'éligibilité
start "" "%drive_letter%:\setup.exe" /product server
exit /b
```