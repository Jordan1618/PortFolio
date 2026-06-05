## Step 1 : Configure the Session

- A clean session need to understand that :** **there is no root, disabled by default**, so I have to use sudo -i and a classic apt update & apt upgrade
- After we set up the **UFW (FireWall) and blocked all ports**, we only authorized the 22/80/443 we could install caddy to reverse proxy the HTTP/S to prevent intrusion.
- And finally I setup the vm inside our network so I made the right configuration, and you know why i can't tell you more there, privacy purposes.

## Step 2 : Configure Docker + Dir

- apt install curl && apt update -y
  curl -fsSL https://get.docker.com | sh
  usermod -aG docker $USER
  docker --version && docker compose version
	- R = Install docker and set up permissions to the user

- mkdir -p ~/n8n-stack
   cd ~/n8n-stack
	-  R = Create n8n dir

- mkdir -p ~/n8n-stack
   cd ~/n8n-stack
	-  R = Create n8n dir

- vim docker-compose.yml
- The config :
	- services:
		  postgres:
		    image: postgres:16-alpine
		    container_name: postgres
		    restart: unless-stopped
		    environment:
		      POSTGRES_USER: X
		      POSTGRES_PASSWORD: You won't have it
		      POSTGRES_DB: X
		    volumes:
		      - postgres_data:/X/X/X/X/X
		  n8n:
		    image: n8nio/n8n:latest
		    container_name: n8n
		    restart: unless-stopped
		    ports:
		      - "5678:5678"
		    environment:
		      DB_TYPE: postgresdb
		      DB_POSTGRESDB_HOST: postgres
		      DB_POSTGRESDB_DATABASE: X
		      DB_POSTGRESDB_USER: X
		      DB_POSTGRESDB_PASSWORD: You won't have it too
		      N8N_ENCRYPTION_KEY: Sorry Too Late
		    depends_on:
		      - postgres
		    volumes:
		      - n8n_data:/X/X/X
		volumes:
		  postgres_data:
		  n8n_data: