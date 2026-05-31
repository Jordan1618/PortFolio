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
- Managed by ICANN (Internet Corporation for Assigned Names and Numbers) inside the IANA division (Internet Assigned Numbers Authority)
- Two kind of TLD : The ccTLD (Country Code TLD) and the gTLD (Generic TLD) 
- It has a very-high cost for an isolated user. Reachable only for big company.





**10) Each layer has its own area of vulnerability and its common attacks :**

|**Layer**|**Role**|**Primary Attack Vectors**|
|---|---|---|
|**Domain (TLD/DNS)**|Identifier / Directory|**DNS Hijacking**, **DNS Spoofing**, **Domain Shadowing**, **Typosquatting**.|
|**Hosting / Network**|Physical Infrastructure|**DDoS** (Distributed Denial of Service), **Saturation attacks**, Hypervisor-level breaches.|
|**Web Server**|The Doorkeeper|**Server software exploits** (e.g., Apache/Nginx), **Misconfiguration** (e.g., Directory Listing), **Brute-force attacks** (SSH/FTP).|
|**Protocols**|Communication Rules|**Man-in-the-Middle (MitM)** (if SSL is misconfigured), **SSL Stripping**, **Session hijacking**.|
|**Code / CMS**|Business Logic|**Injections** (SQLi, XSS, RCE), **Vulnerable plugins/extensions**, **Local File Inclusion (LFI)**.|
|**Database**|Archives|**SQL Injection (SQLi)**, **Data exfiltration** (Dumping), **Corruption attacks**.|
