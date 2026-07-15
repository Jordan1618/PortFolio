@echo off
title Configuration Registre - Passage Force Windows 11
chcp 64001 >nul

:: Vérification et demande automatique des droits Administrateur
fltmc >nul 2>&1 || (
  echo [INFO] Activation des droits administrateur requis...
  powershell -nop -c "start cmd -args '/d/x/r_\"\"%~f0\"\" %*' -verb runas" && exit /b
)

cls
echo ===================================================================
echo   MODIFICATION DES CLÉS REGISTRE (Forcer_Le_Passage_Windows11.bat)
echo ===================================================================
echo.
echo [1/2] Création du dossier MoSetup et injection des valeurs...

:: Chemin de la clé cible (sous HKLM\SYSTEM\Setup comme sur votre capture)
set "target_key=HKLM\SYSTEM\Setup\MoSetup"

:: Ajout des valeurs DWORD (32 bits) fixées à 1 pour sauter les blocages
reg add "%target_key%" /v "AllowUpgradesWithUnsupportedTPMOrCPU" /t REG_DWORD /d 1 /f >nul
reg add "%target_key%" /v "SkipTPMCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_key%" /v "SkipCPUCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_key%" /v "SkipSecureBootCheck" /t REG_DWORD /d 1 /f >nul
reg add "%target_key%" /v "SkipRAMCheck" /t REG_DWORD /d 1 /f >nul

echo [2/2] Opération réussie ! Le Registre est configuré.
echo.
echo ===================================================================
echo  RAPPEL POUR LA SUITE :
echo ===================================================================
echo  1. Ouvrez votre Éditeur de Registre et appuyez sur F5 : 
echo     le dossier 'MoSetup' est maintenant visible sous 'Setup'.
echo  2. Déconnectez votre PC d'Internet (Wi-Fi ou câble).
echo  3. Montez votre ISO Windows 11 25H2 et lancez 'setup.exe'.
echo ===================================================================
echo.
pause
exit /b