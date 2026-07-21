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
4) The , means "and take this thing too"
5) ``-ErrorAction SilentlyContinue`` are standing for "if an error happens, continue without polluting"
6) The | transfers all outputs on his left into inputs on its right
7) ``Where-Object`` is a filter for a list and we use a {...} (script block) ; $_ is used to name the current read file ; .DisplayName means (.) go look for the DisplayName (variable or a name of line) ; -like is for "is it containing that ?" and "ASP.NET Core*" checks if the name corresponds like Smthg or not + The name + Smthg or not
8) ``Select-Object`` means for each object, prints only these information : X,Y,Z . 
9) DisplayName and Version are easy to understand. InstallLocation & UninstallString are for the file location and the uninstall command.

- Nb : It can be declined in many form by changing a filter or an object or a regedit path

# **3) Checking whether the file exists ?**

- ``Test-Path "C:\Program Files\dotnet\shared\Microsoft.AspNetCore.App"``
	``Get-ChildItem "C:\Program Files\dotnet\shared\Microsoft.AspNetCore.App" -ErrorAction SilentlyContinue``

1) Test-Path is a Boolean that send True or False
2) Get-ChildItem is to show everything inside

- Other example :
- ``Test-Path "C:\Windows\System32\msvcr100.dll"``