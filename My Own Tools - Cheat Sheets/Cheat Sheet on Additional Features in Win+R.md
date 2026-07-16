# **1) .NET Framework & Old / Rarely Used Tools

- .NET Framework 3.5 : An older toolkit that some apps still need to run. Nb : ".NET Framework" is a set of ready-made building blocks that many Windows apps are built on. Version 3.5 also includes versions 2.0 and 3.0 inside it. Some old software was built for this old version and won't run without it, so Windows lets you turn it on when needed.
    
- .NET Framework 4.8 Advanced Services : Extra developer pieces for a newer version of the same toolkit. Nb : This adds extra parts of .NET 4.8 (a newer version than 3.5) that are only needed if a program specifically asks for them, like some Windows Communication Foundation (WCF) services used to let programs talk to each other over a network.
    
- Legacy Components : Very old Windows features kept only for compatibility. Nb : This mostly contains DirectPlay, an old system from the early 2000s that let some games connect to each other over a network. Almost nothing uses it today, but Windows keeps it as an option for very old software.
    
- MultiPoint Connector : A tool for connecting several computers to one shared PC. Nb : This is used in the "Windows MultiPoint Server" setup, where several people can use the same physical computer at the same time with separate monitors, keyboards and mice. Very niche, mostly seen in schools.
    

# **2) Virtualization & Containers

- Hyper-V : Microsoft's tool to run other operating systems inside your computer. Nb : Hyper-V is a "hypervisor" — a program that creates virtual computers (VMs) inside your real one. Each VM behaves like its own separate PC with its own operating system, but they all share your one physical machine.
    
- Windows Hypervisor Platform (WHP) : A lighter piece that lets other virtualization tools use Hyper-V's engine. Instead of giving you the full Hyper-V manager, it just exposes the underlying engine so other programs (like some emulators) can use hardware virtualization without needing the whole Hyper-V setup.
    
- Virtual Machine Platform : The base engine that WSL2 and Windows Sandbox run on. Nb : This is a smaller virtualization layer, separate from the full Hyper-V manager. Windows Subsystem for Linux version 2 (WSL2) and Windows Sandbox both need this turned on to work, even if you never touch Hyper-V directly.
    
- Windows Sandbox : A temporary, throwaway mini-Windows for testing risky files or apps. Nb : When you open Windows Sandbox, it creates a brand new, clean, empty Windows environment. Anything you do inside it (install a suspicious app, open a weird file) disappears completely once you close it, so your real computer is never affected.
    
- Containers : Lets Windows run small, isolated app environments instead of full virtual machines. Nb : A container is lighter than a full virtual machine. Instead of simulating an entire separate computer, it just isolates one app and what it needs, while still sharing the same Windows underneath. This is the technology Docker uses on Windows.
    
- Container Server : Extra background components needed to run and manage containers. Nb : This adds server-side pieces that let Windows fully create, run, and manage containers, working together with the main "Containers" feature above.
    
- Host Guardian Service (HGS) : Extra protection for virtual machines, even if the host gets hacked. Nb : HGS stands for Host Guardian Service. It's an advanced Hyper-V security feature used with "Shielded VMs" — virtual machines that stay encrypted and protected even from an administrator of the physical computer they run on. Mostly used in high-security company environments, not home use.
    
- Microsoft NT Kernel Integration Virtual Device : A small internal driver Hyper-V uses to talk to the main Windows kernel. Nb : "NT" is just the name of Windows' core system architecture (short for "New Technology", from the 1990s). This driver is a low-level connector between Hyper-V and the main Windows kernel and usually turns on automatically when needed — you rarely touch it yourself.
    
- Microsoft NT Kernel Integration VSC Driver : Another small internal Hyper-V connector, used from inside virtual machines. Nb : "VSC" stands for Virtualization Service Client.
    

# **3) Linux & Developer Tools

- Windows Subsystem for Linux (WSL) : Lets you run real Linux directly inside Windows. Nb : WSL stands for Windows Subsystem for Linux. It lets you install an actual Linux system (like Ubuntu) and run Linux commands and programs right inside Windows, without needing a separate computer or a full virtual machine.
    
- Windows Projected File System (ProjFS) : The hidden trick that lets WSL and OneDrive show files without fully downloading them first. Nb : ProjFS stands for Projected File System. It lets a program show you a full list of files and folders that look real, even though the actual content isn't stored on your disk yet — it gets fetched only when you open a file. WSL and OneDrive both use this behind the scenes.
    

# **4) Networking & File Sharing

- Data Center Bridging (DCB) : A networking feature that makes shared storage traffic run more smoothly. It's a set of network rules mainly used in servers to make sure storage traffic (like iSCSI) doesn't get slowed down or interrupted by other network traffic on the same cable. Not relevant for normal desktop use.
    
- Services for NFS : Lets Windows share files with Linux and Unix computers using their native method. Nb : NFS stands for Network File System, the standard way Linux and Unix computers share files with each other. This feature lets a Windows computer join in and share or access files using that same method, instead of Windows' own sharing system.
    
- Simple TCP/IP Services : Very old, basic network testing tools. Nb : TCP/IP stands for Transmission Control Protocol / Internet Protocol, the basic set of rules that let computers talk to each other over a network (the internet runs on it). This specific feature just adds a few ancient test services like "echo" and "daytime" that were used decades ago to check if a network connection worked. Basically never needed today.
    
- SMB Direct : A faster version of Windows file sharing, using special network hardware. Nb : SMB stands for Server Message Block, the protocol Windows normally uses to share files and printers over a network. "SMB Direct" is a faster version of it that uses RDMA-capable network cards to move data with less CPU effort — mostly useful for fast storage servers, not normal home use.
    
- SMB 1.0 / CIFS File Sharing Support : The old, unsafe version of Windows file sharing. Nb : CIFS stands for Common Internet File System — these are two names for the same old file-sharing protocol. This version is known for a serious security flaw that the WannaCry virus used to spread. It should stay off unless you truly need to connect to an old device (like an old NAS or printer) that only supports it.
    
- Telnet Client : Connects to old network devices, but with no encryption. Nb : Telnet is an old, very simple way to remotely connect to another device and type commands into it. It sends everything, including passwords, as plain readable text over the network, which makes it unsafe. SSH (Secure Shell) is the modern, encrypted replacement and should be used instead whenever possible.
    
- TFTP Client : A very simple, no-login way to transfer files, often used with network hardware. Nb : TFTP stands for Trivial File Transfer Protocol. It's a stripped-down file transfer method with no usernames or passwords, often used to load configuration files onto routers, switches, or other network equipment during setup.
    
- Remote Differential Compression (RDC) API Support : An old technology for syncing only the changed parts of a file. Nb : RDC stands for Remote Differential Compression. Instead of resending a whole file when it changes, RDC only sends the small parts that actually changed, saving network bandwidth. It was used by some old file-replication systems (like DFS Replication) and is rarely relevant today.
    

# **5) Identity & Directory Services

- Active Directory Lightweight Directory Services (AD LDS) : A small, standalone version of Active Directory. Nb : Active Directory (AD) is Microsoft's system for managing users, computers, and permissions across a company network, usually tied to a "domain." AD LDS is a lighter version that an individual app can use to keep its own simple directory of users or objects, without needing a full company domain behind it.
    
- Windows Identity Foundation 3.5 (WIF) : An old toolkit that helped apps handle logins and identity checks. Nb : WIF stands for Windows Identity Foundation. It's an older framework that let developers build apps which could check "who is this user" using standard identity tokens, instead of building their own login system from scratch. Mostly needed only by older .NET applications today.
    

# **6) Printing, Documents & Web

- Microsoft Print to PDF : Lets you "print" any document straight into a PDF file. Nb : PDF is a very common file type that looks the same on any device. This feature adds a virtual "printer" that, instead of printing on paper, saves the document as a PDF file on your computer.
    
- Internet Information Services (IIS) : Windows' own built-in web server software. Nb : IIS stands for Internet Information Services. It's Microsoft's software for hosting websites and web applications, similar in purpose to Apache or Nginx on Linux. You need this turned on if you want your computer to serve a website or web app locally.
    
- IIS Hostable Web Core : A stripped-down version of IIS that another app can run inside itself. Nb : This lets a regular application include IIS's web-serving engine directly inside itself, instead of relying on the full separate IIS service running in the background. Used mostly by developers building their own hosting tools.
    
- Microsoft XPS Document Writer : An older alternative to PDF, made by Microsoft. Nb : XPS stands for XML Paper Specification, Microsoft's own document format created to compete with PDF. Like "Print to PDF," this adds a virtual printer, but it saves the file as an XPS document instead. Rarely used today since PDF became the standard.
    
- Print and Document Services : The main components that let Windows manage printers and scanners. Nb : This is a group feature that bundles together printer sharing, printer drivers, and document scanning tools, letting a Windows computer act as a print server or manage shared printers for other computers on the network.
    
- Microsoft Message Queuing (MSMQ) Server : An old system that lets apps send messages to each other reliably. Nb : MSMQ stands for Microsoft Message Queuing. It lets one program send a message to another program even if that other program is temporarily offline — the message just waits in a "queue" until it can be delivered. Used by some older enterprise business applications.
    
- Windows Process Activation Service (WAS) : Helps IIS also handle non-website technical services. Nb : WAS stands for Windows Process Activation Service. Normally IIS only responds to web (HTTP) requests. WAS extends that so IIS can also host other types of technical services (like WCF services using other network protocols), not just websites.
    
- Windows TIFF iFilter : Lets Windows Search read text inside TIFF image files. Nb : TIFF stands for Tagged Image File Format, a common type of image file often used for scanned documents. They show up properly in search results.
    

# **7) Media, Security & Misc

- Media Features : The core components that let Windows play audio and video. Nb : This bundles the basic building blocks Windows needs to play music, videos, and use tools like Windows Media Player. Turning it off can break playback in many apps, not just Media Player itself.
    
- Sysmon : A tool that logs detailed activity happening on the computer. Nb : Sysmon (short for "System Monitor") is a monitoring tool from Microsoft's Sysinternals suite. It records detailed logs of things like which programs started, what network connections they made, and what files changed — very useful for security monitoring and matches closely with what you already do with your logging stack.
    
- Device Lockdown : Restricts a computer to only running one specific app. Nb : This is used to set up "kiosk mode," where a computer (like a reception desk screen or public terminal) is locked down to show and run only one chosen application, blocking access to the rest of Windows.
    
- Work Folders Client : Microsoft's own alternative to OneDrive for syncing work files. Nb : Work Folders is a company-run alternative to OneDrive for Business. Instead of using Microsoft's own cloud, a company can run its own Work Folders server, and this feature is the piece that lets a Windows computer connect to and sync with that private company server.