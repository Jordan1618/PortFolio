## Step 1 : Configure the Session

- A clean session need to understand that :** **there is no root, disabled by default**, so I have to use sudo -i and a classic apt update & apt upgrade
- After we set up the **UFW (FireWall) and blocked all ports**, we only authorized the 22/80/443 we could install caddy to reverse proxy the HTTP/S to prevent intrusion.