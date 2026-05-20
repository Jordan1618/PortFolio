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

- **Passive Threats:** Browser fingerprinting (like screen resolution or window size) can uniquely identify you. So i chose to keep the original size and activate the maximale security especially to prevent risk from :
    
- **Active Threats:** Malicious `.onion` sites can exploit your browser's vulnerabilities to execute scripts, install malware, or phish for data.
    
- **User Error:** Logging into personal accounts (Gmail, Facebook) while using Tor instantly destroys anonymity, because those platforms are linking activities to our identity. 

**4) I asked for advice on staying safe and "touring" the network.** To stay safe as a beginner:

- **Default State:** Never resize the browser window; keep it at the default size to avoid fingerprinting.
    
- **Security Levels:** Set the browser's "Shield" icon to "Safest" to block JavaScript.
    
- **Isolation:** Never download files from unknown sources, as they can contain "beacons" that reveal my true IP address once opened.
    
- **Minimalism:** Use Tor only for reading/research, not for downloading or interactive accounts

**5) I asked how hosting an Onion Service works.** It is fundamentally different from the standard Web:

- **No DNS/IPs:** Onion services don't rely on centralized domain registries or public IP addresses.
    
- **Cryptographic Addresses:** An `.onion` address is a cryptographic hash of the site's public key.
    
- **Rendezvous Protocol:** The user and the server connect through a "Rendez-vous point" inside the network. Both parties create encrypted tunnels that meet in the middle, masking the location of both the visitor and the host.

**6) I asked how many nodes exist and how the network stays organized.** It is a decentralized, volunteer-run organism:

- **Volume:** Roughly 7,000–8,000 active nodes globally.
    
- **Directory Authorities:** A small set of trusted servers maintain a "consensus" (an updated list of active nodes). Your browser downloads this list to map the network and build circuits.

**7) I asked if it is dangerous to host a Tor relay.** It depends on my configuration:

- **Middle Relay (Recommended):** It acts as a simple data tunnel. You never see the final destination or content. This is safe, legal, and highly recommended for those wanting to contribute to the network.
    
- **Exit Relay (High Risk):** If i want to be the "final point." If a user does something illegal, it appears to come from my IP address. This can trigger police investigations or ISP complaints.
    
- **Constraint:** Never run an Exit Relay from a home connection. Use a dedicated VPS if I decide to support the network.

**8) I asked to clarify the path of a packet.
The difference is in "visibility":

- **Normal Web:** Your PC $\rightarrow$ ISP $\rightarrow$ Website. Your ISP sees the specific URL and the website sees your home IP.
    
- **Tor Web:** Your PC $\rightarrow$ Guard $\rightarrow$ Middle $\rightarrow$ Exit $\rightarrow$ Website.
    
    - Your ISP only sees a connection to a Tor node (it is blind to your destination).
        
    - The website only sees the Exit node's IP (it is blind to your home location).