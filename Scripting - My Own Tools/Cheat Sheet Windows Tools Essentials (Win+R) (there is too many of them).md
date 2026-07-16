# **1) Very useful all time :

- taskschd.msc : Task Scheduler for automate scripts or some tasks freely of the GPO and only on your local post. I privilege GPO for Schedule on the infrastructure.
- eventvwr.msc : Display all important errors and warnings + informations in details even if not important.
- services.msc : To manage service running in background or not / create one (I prefer the PowerShell in this case)
- perfmon/resmon : To see/monitor what ressources and performances are used or free on disk/memory/CPU/network AND the PID
	Nb : "Get-Process -Id <PID> | Format-List *" in a PowerShell Helps a lot to find more detail on a running task/service/program | IT IS VERY EFFICIENT AND FULFILL MY NEED
	Nb2 : I can go further with Sysinternals (I just have to donwload it on a classic browser and unzip the suite) https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
	Then I exec in admin Process Explorer and enjoy this beautiful tool.
	
  

# **2) How it's working ?

First for .net/.asp and .runtime, we need to get on the official github repo and collect last version before taking the right given url and send a donwload request.
