## **Before entering the subject: The Stack and The Hardware

- One main priority for this project is to have a Free-Open Source Stack
- The stack planned is : LTS Docker + Ollama + API REST + Mistral 22B(Q6) + API Gateway + OpenWebUI Interface + n8n
- On the hardware side it's : Dell Server with ESXI + 256GB RAM + 2 Intel Xeon 2.10Ghz + 8TB Stockage
## **Step 1 : Configure the Server

- A clean session need to understand that :** **there is no root, disabled by default**, so I have to use sudo -i and a classic apt update & apt upgrade
- After we set up the **UFW (FireWall) and blocked all ports**, we only authorized the 22/80/443 we could install caddy to reverse proxy the HTTP/S to prevent intrusion.
- And finally I setup the vm inside our network so I made the right configuration, and you know why i can't tell you more there, privacy purposes.

## **Step 2 : Configure Docker + Dir**

- apt install curl && apt update -y
  curl -fsSL https://get.docker.com | sh
  usermod -aG docker $USER
  docker --version && docker compose version
	- R = Install docker and set up permissions to the user
