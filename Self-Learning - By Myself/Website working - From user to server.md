### Part 1: Fundamentals, Security, and Risks

**1) I have looked into how a website works from A to Z.** It seems to be a stack of successive layers : The TLD (Top-Level-Domain), The DNS (Domain Name System), The Hosting (where data is stored), The Web server (Where it runs), Differents Protocols HTTP / HTTPS / SSL (How it communicates), Code/CMS (What makes everything work) and Database (Where everything is stored)

**2) A right analogy : **
- Domain = The physical address
- DNS = The directory used to find each address
- Hosting = The building or the field
- Webserver = The doorkeeper
- Code/CMS = The furniture inside the home
- Database = The shelves where everything is sorted and retrievable

**3) The TLD :**
- Managed by ICANN (Internet Corporation for Assigned Names and Numbers) through its IANA division (Internet Assigned Numbers Authority)
- Two kind of TLD : ccTLD (Country Code TLD) and gTLD (Generic TLD) 
- It has a very-high cost for an isolated user that want to create one, fortunately it's affordable to rent one. Reachable only for big organisation.
- Works on a system of Root Name Server - Judiciary responsible and high-avaibility unnegociable

**4) The DNS :**
- Based on a request (Query), the DNS Resolver (ISP or Google Service), Talks to the Root Server, that talk to the TLD servers, that talk to the Authoritative Name Server that gives the right IP Address
- In a DNS we got many register : 
	- Register A = For Ipv4
	- Register AAAA = For Ipv6
	- Register CNAME = For one domain name to another
	- Register MX = For email
- To prevent about DNS Hijacking (Attack that corrupt DNS and redirect each flux to malicious website) there is a DNSSEC (Domain Name System Security Extension)

**5) The Hosting :**
- Two distinct approaches : The physical server and the VPS one (Virtual Private Server = Cloud)
- 3 main characteristics : 
1) Accesibility 24/7, linked to internet and cooled. 
2) Connectivity and bandwidth
3) Physical Security like unauthorized physicial access or DDoS

**6) The Connexion between the user and the server :**
- Before sending any data, they have to agree about a correct communication protocol. The famous TCP by Three-Way-Handshake.
- Three steps way : The SYN = the firts packet to initiate ; The SYN-ACK = The answer to the request by the server ; The ACK = User accept to synchronise and connect.
- Another security step required for HTTPS, the TLS Handshake(Transport Layer Security) = User and server agree on a digital certificate + encryption algorithm + generate a symmetric session key

**7) The Webserver :**
- Link the HTTP Protocol outside him, to its own data. 
- Most know software : Apache, Nginx, LiteSpeed/ IIS
- It works on a hearing process on the port 80 and 443. He received an ask form the brower. If it's a static file, he send, otherwise it's a script and send it to a translater/executer like PHP or Python.
- He communicates its error with correspond code : 200 (ok), 404 (no file found), 500 (code problem)
- For HTTPS


**10) Each layer has its own area of vulnerability and its common attacks :**

|**Layer**|**Role**|**Primary Attack Vectors**|
|---|---|---|
|**Domain (TLD/DNS)**|Identifier / Directory|**DNS Hijacking**, **DNS Spoofing**, **Domain Shadowing**, **Typosquatting**.|
|**Hosting / Network**|Physical Infrastructure|**DDoS** (Distributed Denial of Service), **Saturation attacks**, Hypervisor-level breaches.|
|**Web Server**|The Doorkeeper|**Server software exploits** (e.g., Apache/Nginx), **Misconfiguration** (e.g., Directory Listing), **Brute-force attacks** (SSH/FTP).|
|**Protocols**|Communication Rules|**Man-in-the-Middle (MitM)** (if SSL is misconfigured), **SSL Stripping**, **Session hijacking**.|
|**Code / CMS**|Business Logic|**Injections** (SQLi, XSS, RCE), **Vulnerable plugins/extensions**, **Local File Inclusion (LFI)**.|
|**Database**|Archives|**SQL Injection (SQLi)**, **Data exfiltration** (Dumping), **Corruption attacks**.|
 