# **1) What I wanted to do :**

I wanted to turn on Sysmon to log more details about what happens on my PC (which programs start, etc.), just to test what it changes. I found "Sysmon" as a checkbox in "Turn Windows features on or off" and ticked it. But it didn't work right away like a normal installed program.

Problems I got : Get-Service sysmon64 said "no service found with that name" Get-WinEvent on the Sysmon log said "no matching log found" I also had an old Sysmon64.exe file sitting unused on my Desktop, which confused me even more

# **2) What was really happening :**

Ticking the checkbox only turns on the option. It does NOT start the program by itself, you still need one extra command. Also, I learned Windows 11 now comes with its own built-in copy of Sysmon (in `C:\Windows\System32\sysmon.exe`). This is different from the Sysmon file you download yourself from Microsoft's Sysinternals website.

I checked where the Sysmon files were on my PC :

powershell

```powershell
Get-ChildItem -Path C:\ -Filter "sysmon*.exe" -Recurse -ErrorAction SilentlyContinue -Force
```

Result : one copy already built into Windows (System32), and one older copy I had downloaded myself on my Desktop.

I installed the built-in one properly (PowerShell, as Administrator) :

powershell

```powershell
sysmon.exe -accepteula -i
```

Nb : `-accepteula` just skips the license popup. `-i` means "install". No need to move to a specific folder first since System32 is always accessible.

# **3) Checking if it's worked :**

powershell

```powershell
Get-Service sysmon
```

Result : Status "Running" — it works.

Reading the last 10 log entries (short view) :

powershell

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

Reading one full entry with all details (the useful part : command used, file hash, which program started it, which user) :

powershell

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 1 | Format-List *
```

Showing only "a program started" events (Event ID 1), the last 5, in full :

powershell

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Where-Object Id -eq 1 | Select-Object -First 5 -ExpandProperty Message
```

# **4) What I understood from reading the logs :**

- By default, Sysmon only logs two things : when a program starts (ID 1) and when it closes (ID 5). It doesn't log network activity, registry changes, or other useful things yet
- Each entry still gives a lot of detail : the exact command used, a file hash to check if the program is trustworthy, which program launched it, and which Windows user did it
- Reading logs one by one doesn't work well : one simple action (like Obsidian checking my Git folder) creates 5+ log entries at once, and it's all normal, harmless activity
- Raw logs are meant to be filtered first (with a config file), not read one by one in a console

# **5) How to remove it (since I'm just testing) :**

powershell

```powershell
sysmon.exe -u
```

Nb : `-u` uninstalls it — removes the service, the driver, and the log channel, like it was never there.

# **6) What to do next  :**

- Set up a better Sysmon config file to ignore normal daily noise (Git, Obsidian, browser) and only keep the important stuff (suspicious commands, strange network connections, programs starting from weird folders)
- Learn to read the logs better, maybe with a simple viewer instead of raw console output