## How It's working ?

**1) It's a Client-Server architecture

- Usually it's a TCP or UDP port 3389.

**2) The connexion path :

- 1) The connexion : each part exchanges its layer of security, encryption protocol and what is supported or not.
- 2) Multi Channel : open a different channel for each flux (keyboard, mouse, audio, ...)
- 3) Authentification : request of IDs to grant the access
- 4) Session : Once logged, the server starts loading gui towards the client.

**3) How data is travelling :

- 