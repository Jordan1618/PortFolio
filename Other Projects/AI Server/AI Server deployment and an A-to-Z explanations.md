## **Before entering the subject: The Stack and The Hardware

- One main priority for this project is to have a Free-Open Source Stack
- The stack thinked is : LTS Docker + Ollama + API REST + Mistral 22BQ6 + API Gateway + OpenWebUIInterface CHATGPT like n8n
- In the hardware side is : Dell Server with ESXI + 218Go + 
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
