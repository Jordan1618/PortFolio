```batch
@echo off
:: Encodage UTF-8 pour gérer correctement les accents dans la console
chcp 65001 >nul
cls

echo =================================================================
echo   SCRIPT DE CONFIGURATION GLOBALE ET NETTOYAGE SYSTÈME
echo =================================================================
echo.

:: =================================================================
:: ETAPE 1 : ÉLÉVATION DES PRIVILÈGES AUTOMATIQUE
:: =================================================================
echo [INFO] Validation des privilèges administrateur...
net session >nul 2>&1
if %errorLevel% == 0 (
    echo [SUCCÈS] Le script est exécuté avec les privilèges requis.
) else (
    echo [ATTENTION] Droits insuffisants. Tentative d'élévation automatique...
    powershell -Command "Start-Process -FilePath '%0' -Verb RunAs"
    exit /b
)
echo.

echo -----------------------------------------------------------------
echo [ATTENTION] Le script va exécuter toutes les étapes automatiquement.
echo Appuyez sur une touche pour LANCER le processus global...
pause >nul

:: =================================================================
:: ETAPE 2 : NETTOYAGE FORCE ET COMPLET D'ATERA
:: =================================================================
cls
echo =================================================================
echo [ÉTAPES EN COURS] VEUILLEZ PATIENTER...
echo =================================================================
echo.
echo -----------------------------------------------------------------
echo [PROGRÈS] Étape 2 : Suppression et nettoyage des restes d'Atera
echo -----------------------------------------------------------------
echo [INFO] Arrêt des processus Atera en cours...

taskkill /F /IM "AteraAgent.exe" >nul 2>&1
taskkill /F /IM "AgentPackage.exe" >nul 2>&1
taskkill /F /IM "MonitoringService.exe" >nul 2>&1

echo [INFO] Arrêt et suppression des services Windows Atera...
net stop "AteraAgent" >nul 2>&1
sc delete "AteraAgent" >nul 2>&1

echo [INFO] Purge des fichiers et dossiers résiduels...
if exist "%programfiles%\ATERA Networks" rmdir /s /q "%programfiles%\ATERA Networks"
if exist "%programfiles(x86)%\ATERA Networks" rmdir /s /q "%programfiles(x86)%\ATERA Networks"
if exist "%programdata%\ATERA Networks" rmdir /s /q "%programdata%\ATERA Networks"

powershell -Command "Stop-Process -Name 'AteraAgent', 'AgentPackage', 'MonitoringService' -Force -ErrorAction SilentlyContinue" >nul 2>&1
powershell -Command "Remove-Item -Path 'C:\Program Files\ATERA Networks' -Recurse -Force -ErrorAction SilentlyContinue" >nul 2>&1
powershell -Command "Remove-Item -Path 'C:\Program Files (x86)\ATERA Networks' -Recurse -Force -ErrorAction SilentlyContinue" >nul 2>&1

echo [INFO] Nettoyage des clés de Registre Atera...
reg delete "HKLM\SOFTWARE\ATERA Networks" /f >nul 2>&1
reg delete "HKLM\SOFTWARE\WOW6432Node\ATERA Networks" /f >nul 2>&1
reg delete "HKCU\Software\ATERA Networks" /f >nul 2>&1
reg delete "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\AteraAgent" /f >nul 2>&1

echo [OK] Composants Atera retirés du système.
echo.

:: =================================================================
:: ETAPE 3 : NETTOYAGE SÉLECTIF DU DOSSIER WINDOWS UPDATE
:: =================================================================
echo -----------------------------------------------------------------
echo [PROGRÈS] Étape 3 : Nettoyage ciblé du dossier WindowsUpdate
echo -----------------------------------------------------------------
echo [INFO] Purge des restrictions dans la clé Policies\WindowsUpdate...

reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /f >nul 2>&1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /f >nul 2>&1

reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "NoWindowsUpdate" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "SettingsPageVisibility" /t REG_SZ /d "show:windowsupdate" /f >nul 2>&1
echo [OK] Nettoyage du dossier Windows Update effectué (Blocages retirés).
echo.

:: =================================================================
:: ETAPE 4 : CONFIGURATION DU SOUS-DOSSIER AU (AUTOMATIC UPDATES)
:: =================================================================
echo -----------------------------------------------------------------
echo [PROGRÈS] Étape 4 : Configuration et planification dans le dossier AU
echo -----------------------------------------------------------------
echo [INFO] Application des clés de planification...
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ElevateNonAdmins" /t REG_DWORD /d 1 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ElevateNonAdmins" /t REG_DWORD /d 1 /f >nul 2>&1

reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "AUOptions" /t REG_DWORD /d 4 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ScheduledInstallDay" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "ScheduledInstallTime" /t REG_DWORD /d 20 /f >nul 2>&1

reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "NoAutoUpdate" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "AUOptions" /t REG_DWORD /d 4 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ScheduledInstallDay" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "ScheduledInstallTime" /t REG_DWORD /d 20 /f >nul 2>&1

reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /v "ExcludeWUDriversInQualityUpdate" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\AU" /v "AllowMUUpdateService" /t REG_DWORD /d 1 /f >nul 2>&1

echo [OK] Planification à 20h00 et droits utilisateurs configurés dans AU.
echo.

:: =================================================================
:: ETAPE 5 : ACTUALISATION ET REDÉMARRAGE DES SERVICES
:: =================================================================
echo -----------------------------------------------------------------
echo [PROGRÈS] Étape 5 : Application et actualisation des services
echo -----------------------------------------------------------------
echo [INFO] Actualisation des politiques locales (gpupdate)...
gpupdate /force

echo.
echo [INFO] Redémarrage du service Windows Update...
net stop wuauserv >nul 2>&1
net start wuauserv >nul 2>&1

echo.
if %errorLevel% == 0 (
    echo [OK] Le service Windows Update a été actualisé et redémarré.
) else (
    echo [ATTENTION] Erreur lors du redémarrage du service. Un redémarrage PC résoudra cela.
)
echo.
echo Opérations terminées. Passage au résumé...
timeout /t 2 >nul

:: =================================================================
:: FIN DU SCRIPT SUCCÈS
:: =================================================================
cls
echo =================================================================
echo   TOUTES LES OPÉRATIONS ONT ÉTÉ EXÉCUTÉES !
echo =================================================================
echo Les restrictions ont été levées, Atera a été supprimé, et la
echo planification automatique (20h) est active.
echo.
echo [CONSEIL] Un redémarrage du PC est fortement recommandé.
goto FIN

:FIN_ERREUR
echo.
echo =================================================================
echo   LE SCRIPT A ÉCHOUÉ
echo =================================================================
echo Une erreur critique est survenue lors de l'exécution.
echo.

:FIN
echo.
echo [INFO] Fin du script. Appuyez sur une touche pour fermer cette fenêtre.
pause >nul
```