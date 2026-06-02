### Story and Explication

**1) Where Dos comes from : **
- At the very beginning, WIndows was only a CLI and the textual OS was MS-DOS (Microsoft Disk Operating System) in 80's-90's.It was before the mouse and the keyboard exist.
- Now the real Dos doesn't exist anymore it has been replaced by the cmd.exe, the usual CLI.
- The .bat extension comes from the Batch language. It runs successive Dos commands in a single script.

**2) SNMP : **
- SNMP = Simple Network Management Protocol / use for SOC purposes like supervision, managing inventory, real-time incident detection.
-  3 Step-working :
	1) The Manager = Supervision Console (Zaabix, PRTG, Centreon). It interrogates devices and centralize data to display graphics.
	2) The Agent = The service actived in each device and return data requested by the manager
	3) The Managed Device = The physical device or virtual one which cares the agent
- The OID (Object Identifier) is the translater between the manager and the agent. Each piece of data has its own address made of points and digits succession, each is universal. That's the OID.
- The MIB (Management Information Base) is a dictionnary that contains each OID-text translation 