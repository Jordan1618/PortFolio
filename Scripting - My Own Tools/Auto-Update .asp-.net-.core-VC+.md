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

- Added an email notification (via Brevo's API) that only triggers when something goes wrong, so I don't need to check every computer/server manually.
- Added a restart-pending flag, for when some updates need a reboot to fully apply, and that is easy to implement across many machines.
- Added cleanup of old runtime/SDK folders after a successful update, so versions don't stack like useful folders wasting space and storage over time.