# **1) The concept and the problem :

I have a lot of computers and servers that need VC++ or different .asp/.net/.core dependencies and each time I have to download them manually. It's long and can be automated. So I wanted to learn how to make that to save that time and understand more the structure of PowerShell script

# **2) How it's working ?

First for .net/.asp and .runtime, we need to get on the official github repo and collect last version before taking the right given url and send a donwload request.

For the VC++ Redist, we have to use the permanent url of Microsoft and an API key to download the last version and updates it.

The path of the script :
- Variables defining
- Internet test
- Download VC++
- Get last versions of . files and download before comparing with installed
- Clear resume


The Temp variable act as a temporary place to download files before deleting them.
# **3) The total script (in case you need) :