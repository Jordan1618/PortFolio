```powershell
# ==============================================================================================
# Setup-Secret - ONE-TIME setup script
# Run this manually, once, as Administrator, on each machine that needs to read this secret at
# runtime (e.g. a script running under SYSTEM via Task Scheduler).
#
# It stores a secret value (API key, password, token...) encrypted with DPAPI in LocalMachine
# scope, so :
#   - The secret is NEVER stored in plain text anywhere on disk.
#   - It can be decrypted by ANY account on THIS machine (SYSTEM included), but is USELESS if
#     copied to another machine (DPAPI LocalMachine keys are bound to the machine that created
#     them).
#   - Any non-secret identifier (like a key ID or username) should stay in plain text in your
#     own script's config - only the actual secret value goes through this encrypted file.
#
# Re-run this script any time you rotate the secret - it simply overwrites the file.
# ==============================================================================================

# Step 1 : load DPAPI. Not autoloaded by default in either PowerShell edition (5.1 or 7+).
Add-Type -AssemblyName System.Security

# Step 2 : make sure this is running as Administrator. Needed for DPAPI LocalMachine scope
# and for the folder lockdown further down.
$CurrentIdentity  = [Security.Principal.WindowsIdentity]::GetCurrent()
$CurrentPrincipal = New-Object Security.Principal.WindowsPrincipal($CurrentIdentity)
$IsAdmin          = $CurrentPrincipal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
if (-not $IsAdmin) {
    Write-Host "[ERROR] This script must be run as Administrator (DPAPI LocalMachine scope and the ACL lockdown both require it)." -ForegroundColor Red
    exit 1
}

# Step 3 : configuration. Must match the values used in the script that reads the secret back.
$SecretFolder = "C:\Scripts\Secrets"
$SecretFile   = Join-Path $SecretFolder "secret.dat"
# Entropy is an extra "pepper" mixed into the DPAPI encryption. It is NOT a secret on its own
# (it's fine that it sits in plain text here and in the reading script) - it just means that
# even someone with full admin access to THIS machine cannot decrypt the file with a generic
# DPAPI call; they'd also need this exact string. Change it to your own value, just keep it
# identical across this script and every script that reads the secret back.
$Entropy = [Text.Encoding]::UTF8.GetBytes("YourProject-YourService-2026")

Write-Host "=======================================================================" -ForegroundColor Cyan
Write-Host " Secret - one-time secure setup for this machine" -ForegroundColor Cyan
Write-Host "=======================================================================" -ForegroundColor Cyan
Write-Host "Machine: $env:COMPUTERNAME" -ForegroundColor Gray
Write-Host ""

# Step 4 : ask for the secret value. -AsSecureString means it never gets printed to the console.
$SecureSecret = Read-Host "Paste the secret value to encrypt" -AsSecureString

# Step 5 : briefly turn it back into plain text, since DPAPI needs raw bytes, not a SecureString.
# The plain copy only exists for the next few lines, then gets wiped in the "finally" block.
$Bstr         = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecureSecret)
try {
    $SecretPlain = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
} finally {
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($Bstr)
}

if ([string]::IsNullOrWhiteSpace($SecretPlain)) {
    Write-Host "[ERROR] No value entered, aborting. Nothing was written." -ForegroundColor Red
    exit 1
}

try {
    # Step 6 : encrypt the secret with DPAPI, tied to this machine.
    $Bytes     = [Text.Encoding]::UTF8.GetBytes($SecretPlain)
    $Protected = [Security.Cryptography.ProtectedData]::Protect(
        $Bytes, $Entropy, [Security.Cryptography.DataProtectionScope]::LocalMachine
    )

    # Step 7 : write the encrypted result to disk. The plain text never touches the disk.
    if (-not (Test-Path $SecretFolder)) {
        New-Item -Path $SecretFolder -ItemType Directory -Force | Out-Null
    }
    [IO.File]::WriteAllBytes($SecretFile, $Protected)

    # Step 8 : lock the folder down so only SYSTEM and local admins can read it. This removes
    # inherited permissions first (so "Users"/"Authenticated Users" lose default read access),
    # then grants read-only back to just the two accounts that need it.
    icacls $SecretFolder /inheritance:r | Out-Null
    icacls $SecretFolder /grant:r "SYSTEM:(OI)(CI)R" "BUILTIN\Administrators:(OI)(CI)F" | Out-Null

    Write-Host "`n[OK] Secret encrypted and stored at: $SecretFile" -ForegroundColor Green
    Write-Host "[OK] Folder ACL restricted to SYSTEM (read) and local Administrators (full control)." -ForegroundColor Green
    Write-Host "`nThis file is now unreadable outside of this machine. You can safely close this window." -ForegroundColor Gray
} catch {
    Write-Host "[ERROR] Failed to encrypt/store the secret: $_" -ForegroundColor Red
    exit 1
} finally {
    # Step 9 : best-effort cleanup of the plaintext copy that briefly existed in memory.
    Remove-Variable -Name SecretPlain -ErrorAction SilentlyContinue
    [GC]::Collect()
}
```