## **1) 2 Kinds of configuration :**

A GPO (Group Policy Object) is used to apply policies to an OU (Organizational Unit : Computers or Users).

This differentiation happens around only one question : Should this thing be done even if there is no user logged in ? 

If it's yes --> Computer Configuration / Else --> User Configuration.

Normally you have to choose between those branches every time and leave the other empty. And it's more secure to, no matter what, identify simply where a potential failure/error could come from. 

Be careful of these 2 common traps :
1) For a same value on the same parameter, the computer's GPO always wins on a user's GPO
2) If a user configuration applies on a group full of computers, it will be silent and useless

## **2) The 3 categories inside each branch :**

#### Software Settings :

Allow to install/uninstall/update software with msi packages. Currently outdated in compared to Intune or SCCM
Always takes effect after restarting computer/user or gpupdate /force

#### Windows Settings :

Various features native of Windows, different from registry keys. It's more like system mechanics.
There are 3 categories inside :
- Scripts : Execution when the computer or user restarts or logs in
- Security Settings : Password policies / Users rights / Firewall or not / Restrictions
- Preferences : Various possibilities (Choose execution once or every time; Values by default can be edited by users or not; Very wide range(registry, mappers, local users, printers, task scheduler; etcetera))

#### Administration Templates

The most used category by far, for sure. This is a succession of .adm/.adml that covers all path to keys in registry. The easiest to updates features or policies. Microsoft even releases NIS2-complaient files ready to be set up in GPO.

## **3) A brief point on OU topic**

OU stands for Organizational Unit. An OU regrou