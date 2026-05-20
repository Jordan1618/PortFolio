### Part 1: Fundamentals, Security, and Risks

**1) I asked how Tor works from A to Z.** It works through a process called "Onion Routing":

- **Three-Layer Encryption:** Your data is protected in three layers of encryption before leaving your PC.
    
- **The Circuit:** Your traffic passes through three nodes: the **Guard** (knows you, not the destination), the **Middle** (knows neither), and the **Exit** (knows the destination, not you).
    
- **Anonymity:** Because each node only knows its immediate neighbor, no single point in the network can map your entire identity to your destination.
	
- **Routing :** I understand the following scheme : My Laptop ->(Tor request, not an clear reasable dns resolution asking) ISP -> Guard node -> Middle Node -> Exit node -> Website
	
- **Complementary point : The "ISP Blindness" (DNS Resolution)** In a normal web request, my laptop asks the ISP: What is the IP of propublica.org? (This is a clear DNS request). 
- In my Tor scheme, my laptop **doesn't ask the ISP for the IP**. Instead, the Tor Browser handles the address resolution through the circuit itself. 
- The ISP never sees a DNS request for propublica.org. It only sees my laptop initiating an encrypted connection to the IP address of the Guard Node.

**2) I asked if using Tor is illegal.** It is not illegal:

- **Legality:** Using Tor is legal in some countries. It is a legitimate tool for privacy, journalism, and protecting sensitive information.
    
- **Responsibility:** While the tool is legal, the usage is subject to the law. Committing crimes, illegal purchases, or accessing prohibited content (download, becoming a bot) via Tor remains a criminal offense.

**3) I asked about the risks to my computer and privacy.** The risks are both technical and behavioral:

- **Passive Threats:** Browser fingerprinting (like screen resolution or window size) can uniquely identify you. So i chose to keep the original size and activate the maximale security especially to prevent autodownload or malveillous javascript execution
    
- **Active Threats:** Malicious `.onion` sites can exploit your browser's vulnerabilities to execute scripts, install malware, or phish for data.
    
- **User Error:** Logging into personal accounts (Gmail, Facebook) while using Tor instantly destroys your anonymity, as those platforms link your activity to your identity.