# **1) Very useful all time :**

- taskschd.msc : Task Scheduler for automate scripts or some tasks freely of the GPO and only on your local post. I privilege GPO for Schedule on the infrastructure.
- eventvwr.msc : Display all important errors and warnings + informations in details even if not important.
- services.msc : To manage service running in background or not / create one (I prefer the PowerShell in this case)
- perfmon/resmon : To see/monitor what ressources and performances are used or free on disk/memory/CPU/network AND the PID
	Nb : " Get-Process -Id "X" | Format-List * " in a PowerShell Helps a lot to find more detail on a running task/service/program | IT IS VERY EFFICIENT AND FULFILL MY NEED
	Nb2 : I can go further with Sysinternals (I just have to donwload it on a classic browser and unzip the suite) https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
	Then I exec in admin Process Explorer and enjoy this beautiful tool.
- compmgmt.msc : To have and control all the previous tools !!! Incredible. This + Process Explorer are game changer in AdminSys field.
  
# **2) Security and Compliancy**

- secpol.msc : Displays all your security policies
- gpedit.msc : Useful for testing locally before making any change
- certmgr.msc : Manages certificates in internal / useful for HTTPS or different access
- wf.msc : Firewall Windows
- lusrmgr.msc : Manages local Users and Groups (except the AD ones)

# **3) General Diagnostics**

- devmgmt.msc : Device manager
- diskmgmt.msc : Disk Manager
- msconfig : System Conf
- msinfo32 : All system Information
- regedit : You know and I know
- optionalfeatures : Enables or disables other features. (Needs it's own Cheat Sheet I guess)
- cleanmgr : To clear unnecessary files

# **4) Very Common and Useful Cli Commands**

- gpupdate /force 
- gpresult /r
- sfc /scannow
- ipconfig /all
- nslookup
- netstat -ano
- ping
- Get-WinEvent

Nb : Of course I know each one of them and use them regularly