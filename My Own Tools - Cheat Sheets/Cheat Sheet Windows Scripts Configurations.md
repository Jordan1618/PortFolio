# **1) Preparation for Script Execution**

 - ``Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`` : Unlock scripts execution for this PowerShell instance. Process is the indicator. 
 - ``cd \\X\X\X\X + .\FileName`` : uses rightfully 

# **2) Looking into the Registry** 

- ``Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*", "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*"
    -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -like "*ASP.NET Core*" } |
    Select-Object DisplayName, DisplayVersion, InstallLocation, UninstallString``

- There are many explanations :
1) Backsticks at the end of the first 3 lines are a pure commentary to say "go next line"
2) ``Get-ItemProperty ``is to select a property of an element
3) ``HKLM + the *`` are for the regedit + a wildcard (taking all elements at this place)
4) 