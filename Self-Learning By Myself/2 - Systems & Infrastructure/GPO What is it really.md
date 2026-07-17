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

The most used category by far, for sure. This is a succession of .adm/.adml that cover every path to keys in the registry. The easiest way to updates features or policies. Microsoft even releases NIS2-compliant files ready to be set up in GPO.

## **3) A brief point on OU and GPO topics**

OU stands for Organizational Unit. An OU groups together, within the AD one or several objects like user accounts, computers, groups or another OU.
It's important to split them into machines groups or users groups.

GPO can only be applied on a Domain/Site/OU and under-OU by an inheritance system. 
For sorting a machine or an OS type we use a security sort or WMI sort

The main rule for priority is LSDOu : Local > Site > Domain > OU.
This is very important for the inheritance and the enforced system. Always document each modification and do it only if it's very important because it's complexify the debug after.

## **4) Good habits to Have :**

1) One GPO = One clear Objective
2) Name every GPO with : Range-DomainConcerned-Description
3) Disable the unuse part of GPO
4) Always testing a GPO on an OU test with 1-2 machines/users
5) Never touch the Default Domain Policy, except for Kerberos or Password policies
6) Always let a short description of "why this GPO"
7) gpresults /h rapport.html to check if every GPO is ready and running or not
8) gpresult /r for CLI check
9) gpupdate /force

