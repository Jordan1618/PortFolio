### Part 1: Fundamentals, Security, and Risks

**1) I asked how Tor works from A to Z.** It works through a process called "Onion Routing":

- **Three-Layer Encryption:** Your data is protected in three layers of encryption before leaving your PC.
    
- **The Circuit:** Your traffic passes through three nodes: the **Guard** (knows you, not the destination), the **Middle** (knows neither), and the **Exit** (knows the destination, not you).
    
- **Anonymity:** Because each node only knows its immediate neighbor, no single point in the network can map your entire identity to your destination.
	
- **Routing :** I understand the following scheme : My Laptop ->(Tor request, not an clear reasable dns resolution asking) ISP -> Guard node -> Middle Node -> Exit node -> Website