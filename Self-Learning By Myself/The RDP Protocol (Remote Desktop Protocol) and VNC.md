## How It's working ?

**1) It's a Client-Server architecture

- Usually it's a TCP or UDP port 3389.

**2) The connection path :

- 1) The connexion : each part exchanges its layer of security, encryption protocol and what is supported or not.
- 2) Multi Channel : opens a different channel for each flux (keyboard, mouse, audio, ...)
- 3) Authentification : requests of IDs to grant access
- 4) Session : Once logged in, the server starts loading gui towards the client.

**3) How data is travelling :

- There is no video flux. The server tells the client to draw picture orders. For larger drawings like a wallpaper, it splits into a multitude of little 64X64 pixel squares. 
- To optimize further. It uses the client's cache for recurring visual items like desktop icon and it only updates what has changed, it keeps the unchanged parts.

**4) Protocols stack, why ?

- A lot of these protocols came from the UIT (ITU International Telecommunication Union) which set in 90's, different norms with the T.120 for screen sharing.
	- The T.128 was designed to allow multiple users to share the same application. Microsoft bought a NetCentric Licence that evolves inside the corporation from NetMeeting into to the current RDP
- The TPKT (Transport Packet) is the translater from the OSI system to the TCP/IP system. Its role is to emulate OSI transport service over a TCP/IP network. 4 bytes in front of each packet to give version number and length 
- VNC is different because it transfers screenshots took constantly.
- TeamViewer and AnyDesk works with their own server that secures connexion. Their protocols are proprietary and similar to VNC but very optimized. 

**5) VNC :

- VNC is based on RFB (Remote Framebuffer) protocol. RFB came from At&t in 90's. VNC is a free open-source application. 
- RFB has a client ressource-less functioning, the server carries the weight. VNC client doesn't need calcul power.
- The Framebuffer is the part of the RAM on the GPU that displays pixels on the screen. It just sees pixels grids.
- The client is sending request to the server to make screen modifications. And this loop is repeat again and again.
- One main weakness is that it doesn't carry sound transmission.