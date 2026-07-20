# **1) What I wanted to do :**

I wanted one script that fixes every alert from my CyberAgent report, except Java which I handle separately, and that runs by itself on any of my machines, whether it's a database server or a normal PC, without me changing anything per machine.

# **2) Making it work on any machine :**

Before touching anything, the script checks if that piece of software is even installed, by looking at the same list of installed programs Windows itself shows in "Add or remove programs". If it's not there, it skips it and moves on, no error. That's what lets one script run on every machine instead of me writing one version per machine.

# **3) One shared folder for every VM, instead of copying files everywhere :**

I used to drop the `.msu` update files, downloaded from Microsoft's update catalog, on each machine myself, one by one. Now I download them once and put them in a shared folder that every machine reads from over the network.

Nb : the script runs under a system account called SYSTEM, not under my own account, and SYSTEM authenticates on the network as the machine's own computer account. I had to give that account read permission on the shared folder. Without it, the folder just looked empty even with files inside.

# **4) Checking what a file really is, not just its name :**

I used to trust the filename to know which update (KB number) it was. That's risky: a wrong or renamed file means the script thinks it installed the wrong thing. Now it reads the update's real identity from inside the file itself, using DISM :

```powershell
dism.exe /Online /Get-PackageInfo /PackagePath:"C:\path\to\file.msu"
```

That identity is written into the file by Microsoft and can't be faked by renaming it. If DISM fails for some reason, the script falls back to reading the KB number from the filename, and flags that specific KB as lower confidence in the log so I know to double check it.

# **5) A wrong file still can't break anything :**

`wusa.exe` checks by itself whether an update actually fits the machine (right Windows version, right architecture) before touching anything. If it doesn't fit, it returns a specific exit code, `2359302`, instead of causing any damage. I taught the script to read that code as "nothing to worry about" instead of an error. Exit code `3010` means it worked but a restart is needed.

# **6) The rest of the software list :**

The other programs, Visual C++ runtime libraries, Windows Defender, VMware Tools, TeamViewer Host, Firefox, etcetera. All follow the same idea: check if it's installed, check the current version, download and install only if a newer one exists. Each one is skipped cleanly if that program isn't on the machine.

# **7) What I decided not to touch automatically :**

For my database software and my backup software, the script only checks and reports what needs updating, it doesn't install anything by itself there. Those are too risky to automate without me checking first, a mistake there could mean lost data or downtime.

# **8) Restarting only at night :**

If a restart is needed, the machine doesn't reboot right away. Instead the script schedules it for the evening using `shutdown.exe`, so the machine stays usable all day.

# **9) Getting an email only when needed :**

I only get an email, sent through Brevo's email API, if something actually went wrong, or if a restart got scheduled. A clean run sends nothing.

# **10) Bug : the shared folder emptied itself after the first VM ran :**

I had 5 `.msu` files in the shared folder, one set per Windows build. The first version of the script moved every processed file into an `Installed\` subfolder once it was done with it, success or not. That's fine for one machine, but on a shared folder read by several VMs it broke everything: the first VM to run treated all 5 files (installing its own, marking the other 4 as "not applicable"), then moved all 5 out. The next VM found an empty folder and never got its own update.

Fix : the script no longer moves or deletes anything from the shared folder. Every VM re-scans the full folder on every run. `Get-HotFix` skips a KB already installed on that machine, and `wusa.exe`'s own applicability check skips a KB that isn't meant for that machine. Each VM only installs what's actually for it, and the file stays there for the others.

# **11) When I killed a run halfway through :**

I hit Ctrl+C twice while `wusa.exe` was mid-install. The script stopped without going through its normal cleanup (removing its lock file). The next run refused to start :

```
[CRITICAL ERROR] Another instance of this script appears to be running (lock file age: 24 minutes).
```

I checked nothing was still running before touching anything :

```powershell
Get-Process wusa -ErrorAction SilentlyContinue
```

Nothing came back, so `wusa.exe` had actually stopped, nothing was mid-install. Then I checked which KB had really made it in before the kill :

```powershell
Get-HotFix -Id KB5099539 -ErrorAction SilentlyContinue
```

The script only treats its own lock file as stale after 2 hours, mine was 24 minutes old, so I removed it by hand :

```powershell
Remove-Item "C:\Users\The\Full\Path\thefile.lock" -Force
```

# **12) Reading an exit code I didn't expect :**

`wusa.exe` returned `-2145124329`, a code my script didn't recognize yet (it only knew `0`, `3010`, `2359302`). Converted it to hex to look it up :

```powershell
'{0:X8}' -f -2145124329
```

Result : `80240017`, which is `WU_E_NOT_APPLICABLE`. Same meaning as `2359302`, just a different code `wusa.exe` can return for the same "not for this machine" situation. Added it as another "not applicable" case instead of letting it show up as an error.

# **13) What I learned :**

- Never trust a filename, check what DISM says is really inside the file when I can.
- A script running as SYSTEM doesn't have my own network access, it needs its own permissions.
- Letting `wusa.exe` catch its own mismatches is safer than trying to guess every case myself.
- A shared folder read by several machines must never be emptied by the first one that runs, every machine needs to see the full set every time.
- A killed run doesn't corrupt anything by itself, but it leaves a lock file behind that I have to clear by hand.