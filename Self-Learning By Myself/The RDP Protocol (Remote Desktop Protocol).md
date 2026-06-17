## How It's working ?

**1) It's a Client-Server architecture

- Usually it's a TCP or UDP port 3389.

**2) The connexion path :

- 1) The connexion : each part exchanges its layer of security, encryption protocol and what is supported or not.
- 2) Multi Channel : open a different channel for each flux (keyboard, mouse, audio, ...)
- 3) Authentification : request of IDs to grant the access
- 4) Session : Once logged, the server starts loading gui towards the client.

**3) How data is travelling :

- There is no video flux. The server tells to the client to draw picture orders. For larger draws like a wallpaper, it splits it to a multitude of little square 64X64 pixels. 
- To optimise again. It uses the client cache recursive visual items like desktop icon and it only changes what changed, it keeps the unchanged parts.

**4) Protocols stack, why ?

- A lot of these protocols came from the UIT (Union International of Telecommunication) that sets in 90's, different norms with the T.120 for screen sharing.
	- The T.128 was designed to allow the multi-use of a same application. Microsoft bought a NetCentric Licence that evolves inside the corporation from NetMeeting to the current RDP
- The TPKT (Transport Packet) is the translater from the OSI system to the TCP/IP system. Its role is to emulate OSI transport service over a TCP/IP network.