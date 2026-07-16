### Story and Explication

**1) Where Dos comes from : **
- At the very beginning, WIndows was only a CLI and the textual OS was MS-DOS (Microsoft Disk Operating System) in 80's-90's.It was before the mouse and the keyboard exist.
- Now the real Dos doesn't exist anymore it has been replaced by the cmd.exe, the usual CLI.
- The .bat extension comes from the Batch language. It runs successive Dos commands in a single script.

**2) SNMP : **
- SNMP = Simple Network Management Protocol / use for SOC purposes like supervision, inventory management, real-time incident detection.

-  3-step architecture :
	1) The Manager = Supervision Console (Zaabix, PRTG, Centreon). It interrogates devices and centralize data to display graphs.
	2) The Agent = The service activated on each device that return data requested by the manager
	3) The Managed Device = The physical device or virtual one which cares the agent

- The OID (Object Identifier) is the data address for the agent. Each piece of data has its own address made of points and digits succession, each is universal. That's the OID.
- The MIB (Management Information Base) is a translation dictionnary that contains each OID-name translation. 

- Some commands (Listening on port UDP 161/162) :
	1) GET = A request sent to an agent for a specific value
	2) GET-NEXT / GET-BULK = A request sent for the next piece of data or a large block of data.
	3) SET = An order from the Manager to an Agent to change a conf
	4) (On porte 162) TRAP / INFORM = An Agent sends to the Manager a critical alert without being asked.

- SNMP Version : The V1 is deprecated, The V2 is a misery of security too but add GET-BULK command and others. The V3 is the only secure because there is an encryption of data and an authentification by Id/Password on SHA/MD5

- How to deploy ? 
	1) On the device, enable or install the service "SNMP".
	2) Configuration of the version, creation of user/password and fills the IP Address of supervision server to authorized to query the device.
	3) On the supervision software : Add the device with its IP Address and the same user/password

- But he is a passive protocol, where a glpi agent is active, collect data and send it to the glpi server (the manager).
- Now SNMP is outdated and had been vanished by Microsoft. He stayed only for some software that needs the UDP port 162 interruption service and Windows Server supervision.

R= Query the ; Device ; Deprecated ; Specific value ; Supervision