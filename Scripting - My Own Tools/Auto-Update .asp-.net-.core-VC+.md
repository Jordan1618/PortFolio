# **1) The concept and the problem :

I have a lot of computers and servers that need VC++ or different .asp/.net/.core dependencies and each time I have to download them manually. It's long and can be automated. So I wanted to learn how to make that to save that time and understand more the structure of PowerShell script

# **2) How it's working ?

First for .net/.asp and .runtime, we need to get on the official github repo and collect last version before taking the right given url and send a donwload request.

For the VC++ Redist, we have to use the permanent url of Microsoft and an API key to download the last version and updates it.

The path of the script :
- Variables defining
- Internet test
- Compare and Download VC++
- Get last versions of . files and download before comparing with installed
- Clear resume


The Temp variable act as a temporary place to download files before deleting them.
# **3) The total script (in case you need) :

[Tool --- Automated Updates on AspCoreNetRuntimeVC++](Tool%20---%20Automated%20Updates%20on%20AspCoreNetRuntimeVC++.md)

# **4) The final functioning after I add more features :

I use the following version : [Tool --- Automated Updates V2 On Asp...](Tool%20---%20Automated%20Updates%20V2%20On%20Asp....md)

- **Checks if it's Admin before doing anything** : V1 just ran and could fail halfway if it wasn't allowed to install something. V2 checks first, and stops right away with a clear message if the script wasn't launched as Administrator.
- **Blocks itself from running twice at the same time** : if Task Scheduler retries the script while it's still running, V1 could end up with two copies working at once. V2 creates a small "lock" file at the start and checks for it, so a second copy won't start on top of the first.
- **Now also updates the .NET SDK** : V1 only checked the runtimes (the "shared" folder). V2 adds a third check for the SDK folder, using the same logic (compare, download, install, clean old versions).
- **Sends an email when something goes wrong** : V1 only showed a summary in the console, so I had to open every VM to see if there was a problem. V2 sends an email (via Brevo) with the machine name, its IP, and what went wrong, but only when needed.
- **Keeps a raw technical note for each error** : V1 only wrote a short line in the summary. V2 also saves the exact error, which command caused it, and when, so I can just copy-paste that into a debugging session instead of digging through the log.
- **Knows when a restart is needed** : V1 mentioned it in the summary but didn't track it. V2 keeps a clear yes/no flag for "this machine needs a reboot", shows it at the end, and sends an email about it too, even if there was no real error.
- **Works fine with no one watching** : V1 always waited for a keypress at the end, which is useless (and blocking) when the script runs by itself at night. V2 only waits for a keypress if someone launched it by hand, and closes cleanly on its own the rest of the time

# **5) And how I automated it on my computer :

1) I create an appropriate folder at my Pc's root :
	New-Item -Path "C:\ScriptsJordan" -ItemType Directory -Force
2) cd "C:\ScriptsJordan"
3) I authorized the script execution :
	Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
4
