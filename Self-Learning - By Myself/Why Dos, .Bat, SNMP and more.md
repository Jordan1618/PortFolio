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
- The MIB (Management Information Base) is a dictionnary that contains each OID-name of the device translation 

- Some commands (Listening on port UDP 161/162) :
	1) GET =An Asking to an agent
	2) GET-NEXT / GET-BULK = An Asking for a piece of data or all pieces
	3) SET = An Ordering from the Manager to an Agent
	4) (On porte 162) TRAP / INFORME = An Agent send to the Manager a critical piece of news.

- SNMP Version : The V1 is abandonned, The V2 is a misery of security too but add GET-BULK command and others. The V3 is the only secure because there is an encryption of data and an authentification by Id/Password on SHA/MD5

- How to deploy ? 
	1) On the device, install the service "SNMP".
	2) Configuration of the version, creation of user/password and filling the IP Address of supervision server to receive the asking.
	3) On the supervision software : Add the device with its IP Address and the same user/password

- But he is a passive protocol, where a glpi agent is active, collect data and send it to the glpi server (the manager)