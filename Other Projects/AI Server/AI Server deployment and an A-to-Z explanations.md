## **Before entering the subject: The Stack and The Hardware

- One main priority for this project is to have a Free-Open Source Stack
- The stack planned is : LTS Docker + Ollama + API REST + Mistral 22B(Q6) + API Gateway + OpenWebUI Interface + n8n + LiteLLM
- On the hardware side it's : Dell Server with ESXI + 256GB RAM + 2 Intel Xeon 2.10Ghz + 8TB Storage
## **Step 1 : Configure the Server + SSH + UFW

- Seting up a Raid 5 Replication Storage
- Installing a Ubuntu Resolute Raccoon 26.04 OS in CLI with a Rufus's Bootable USB Key
- Configurating the *ssh key to my main computer with apt install openssh-server -y* + add a *right user with usermod -aG sudo NAME* +  *apt update && apt full-upgrade -y* + apt install *unattended-upgrades && dpkg-reconfigure --priority=low unattended-upgrades*
- ufw default deny incoming
  ufw default allow outgoing
  ufw allow from 192.168.1.0/24 to any port 22 comment 'SSH' sudo ufw allow from 192.168.1.0/24 to any port 3000 comment 'Open WebUI' sudo ufw allow from 192.168.1.0/24 to any port 4000 comment 'LiteLLM Proxy' sudo ufw allow from 192.168.1.0/24 to any port 5678 comment 'n8n Automation' sudo ufw allow from 192.168.1.0/24 to any port 19999 comment 'Netdata Monitoring'
  ufw enable
- ubuntu-report send no
## **Step 2 : Configure Docker + systemd

- We started prerequired for the following steps : sudo apt install -y curl git ca-certificates gnupg lsb-release
	
- We used : 
	 Install -m 0755 -d /etc/apt/keyrings
	 curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
	  -o /etc/apt/keyrings/docker.asc
	  chmod a+r /etc/apt/keyrings/docker.asc
	  
	 *Configure a secure GPG key repository to enforce signed verification during Docker updates.*
	
- After :
	 echo \
	 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
	 https://download.docker.com/linux/ubuntu \
	 $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
	 | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
- 
	 *Install Docker CE + Compose (Docker.io is obsolete and docker CE stands for CommunityEdition) docker-ce = daemon / docker-ce-cli = for commands / containerd.io is the low level manager that communicates with the kernel / docker-build-plugin = allow to build complex architectury / docker-compose-plugin = the go script that start alle the process
	
	 **Docker is a modular system so we have to "craft" our installation**
	 
- Adding the user to Docker's group :
	 apt install util-linux-extra -y
	 usermod -aG docker $USER
	 newgrp docker
	
- mkdir -p /etc/docker ; tee /etc/docker/daemon.json <<'EOF'
	 {
	    "exec-opts": ["native.cgroupdriver=systemd"],
	    "log-driver": "json-file",
	    "log-opts": {
	     "max-size": "20m",
	     "max-file": "3"
	    }
	  }
	 EOF
     *R = Daemon.json creation about logs*
	
- systemctl daemon-reload
  systemctl restart docker
  docker info | grep -i cgroup

- mkdir -p ~/ai-stack
	cd ~/ai-stack
	git init

- docker network create ai-internal
	*Internal network*

- cat > ~/ai-stack/docker-compose.yml <<'EOF'
		services:
		
		  ollama:
		    image: ollama/ollama:latest
		    container_name: ollama
		    restart: unless-stopped
		    networks:
		      - ai-internal
		    volumes:
		      - ollama_data:/root/.ollama
		    deploy:
		      resources:
		        limits:
		          memory: 128g
		
		  litellm:
		    image: ghcr.io/berriai/litellm:main-latest
		    container_name: litellm
		    restart: unless-stopped
		    networks:
		      - ai-internal
		    ports:
		      - "4000:4000"
		    volumes:
		      - ./litellm_config.yaml:/app/config.yaml
		    command: ["--config", "/app/config.yaml", "--port", "4000"]
		    depends_on:
		      - ollama
		    deploy:
		      resources:
		        limits:
		          memory: 1g
		
		  n8n:
		    image: n8nio/n8n:latest
		    container_name: n8n
		    restart: unless-stopped
		    networks:
		      - ai-internal
		    ports:
		      - "5678:5678"
		    volumes:
		      - n8n_data:/home/node/.n8n
		    environment:
		      - N8N_BASIC_AUTH_ACTIVE=true
		      - N8N_BASIC_AUTH_USER=admin
			  - N8N_BASIC_AUTH_PASSWORD=HEPHEPPASTOUCHE
		      - N8N_HOST=localhost
		      - N8N_PORT=5678
		      - GENERIC_TIMEZONE=Europe/Paris
		    deploy:
		      resources:
		        limits:
		          memory: 2g
		
		networks:
		  ai-internal:
		    external: true
		
		volumes:
		  ollama_data:
		  n8n_data:
		EOF
	*R = Configure the following containers by mixing cat > X << 'EOF' 

- cat > ~/ai-stack/litellm_config.yaml <<'EOF'
		model_list:
		  - model_name: mistral
		    litellm_params:
		      model: ollama/mistral
		      api_base: http://ollama:11434
		
		  - model_name: llama3
		    litellm_params:
		      model: ollama/llama3
		      api_base: http://ollama:11434
		
		litellm_settings:
		  num_retries: 3
		  request_timeout: 120
		EOF 
	 *R = Mistral is the mathematical base, ollama the translater from mathematics to the verbal brain and LiteLLM the translater between software applicatons and Ollama*

- Now we are testing and launching them :
	cd ~/ai-stack
	docker compose up -d
	*R = Compose is to select all containers and up to sets them up and the -d tells docker to run its services in background.

- docker exec -it ollama ollama pull mistral
	 *R = Docker exec = executes a command inside a running container / -it is used to enable interactive and terminal / ollama because it's the name of the container / ollama is for the application itself / pull mistral is to install mistrall from the ollama's librairy*

- docker exec -it ollama ollama pull mistral
	 *R = Docker exec = executes a command inside a running container / -it is used to enable interactive and terminal / ollama because it's the name of the container / ollama is for the application itself / pull mistral is to install mistrall from the ollama's librairy*

- curl http://localhost:11434/api/generate \
  -d '{"model":"mistral","prompt":"Bonjour","stream":false}'
	- *R = curl is to send request / -d is to send data in JSON form / The 3 settings especially "stream": false to only give the final sentence, not the stream of continuous words generating*
	
- Mistral model wasn't usefull so we installed Mistral8x22B :
	- docker exec -it ollama ollama run mistral
	- docker exec -it ollama ollama rm mistral
	- cd ~/ai-stack
	- docker compose up -d
	- docker exec -it ollama ollama pull mixtral:8x22B
	- docker exec -it ollama ollama run mistral:8x22B
	- 
- Mistral8x22B model was too slow because we got a lack of GPU, so we try the mistral:8x7 (means on a Meo Architecture 8experts of 7billions of settings) :
	- docker exec -it ollama ollama rm mixtral:8x22b
	- docker exec -it ollama ollama pull mixtral:8x7b
	- docker compose up -d
	- docker exec -it ollama ollama run mixtral:8x7b

## **Step 3 : Configure OpenWebGUI + Modif conf files

- It was a very hard setup to do :
	- In ~/ai-stack/docker-compose.yml :
		````
		```
		cat > ~/ai-stack/docker-compose.yml <<'EOF'
			services:
			
			  ollama:
			    image: ollama/ollama:latest
			    container_name: ollama
			    restart: unless-stopped
			    environment:
			      - OLLAMA_HOST=0.0.0.0:11434
			      - OLLAMA_KEEP_ALIVE=-1
			      - ai-internal
			    volumes:
			      - ollama_data:/root/.ollama
			    deploy:
			      resources:
			        limits:
			          memory: 150g 
			          
			  litellm:
			    image: ghcr.io/berriai/litellm:main-latest
			    container_name: litellm
			    restart: unless-stopped
			    ports:
			      - "4000:4000"
			    environment:
			      - LITELLM_MASTER_KEY=ma-cle-ia-2026
			    networks:
			      - ai-internal
			    volumes:
			      - ./litellm_config.yaml:/app/config.yaml
			    command: ["--config", "/app/config.yaml", "--port", "4000"]
			    depends_on:
			      - ollama
			    deploy:
			      resources:
			        limits:
			          memory: 2g
			
			  open-webui:
			    image: ghcr.io/open-webui/open-webui:main
			    container_name: open-webui
			    restart: unless-stopped
			    ports:
			      - "3000:8080"
			    environment:
			      - OPENAI_API_BASE_URL=http://litellm:4000/v1
			      - OPENAI_API_KEY=ma-cle-ia-2026
			      - WEBUI_AUTH=true
			    networks:
			      - ai-internal
			    volumes:
			      - webui_data:/app/backend/data
			    depends_on:
			      - litellm
			    deploy:
			      resources:
			        limits:
			          memory: 4g
			
			  n8n:
			    image: n8nio/n8n:latest
			    container_name: n8n
			    restart: unless-stopped
			    ports:
			      - "5678:5678"
			    environment:
			      - N8N_BASIC_AUTH_ACTIVE=true
			      - N8N_BASIC_AUTH_USER=admin
			      - N8N_BASIC_AUTH_PASSWORD=password_test_n8n
			      - GENERIC_TIMEZONE=Europe/Paris
			    networks:
			      - ai-internal
			    volumes:
			      - n8n_data:/home/node/.n8n
			    depends_on:
			      - litellm
			    deploy:
			      resources:
			        limits:
			          memory: 4g
			
			networks:
			  ai-internal:
			    external: true
			
			volumes:
			  ollama_data:
			  webui_data:
			  n8n_data:
			EOF 
		````
	-  In ~/ai-stack/litellm_config.yaml :
		- cat > ~/ai-stack/litellm_config.yaml <<'EOF'
		model_list:
			-model_name: mistral-large
		    litellm_params:
		      model: ollama/mixtral:8x7b
		      api_base: http://ollama:11434

		litellm_settings:
		  num_retries: 3   
		  request_timeout: 600 
		  telemetry: false 
		EOF
- Starting and initialisation of the Stack :
	- cd ~/ai-stack
	 docker compose down
	 docker compose up -d
     docker ps
- Testing the UI :
	http://10.0.1.180:3000
- Go to the connexion page of Open WebUI and switch off API Open AI Compatible + Go to API Ollama and put in the URL section : http://ollama:11434

![](Pasted%20image%2020260608172826.png)


## **Step 4 : Creating daemon + Check Different Status Netdata / Prometheus

- We create the daemon to make our stack working even the computer is closed.
```
	- sudo tee /etc/systemd/system/ai-stack.service <<'EOF'
	[Unit]
	Description=AI Stack (Ollama + LiteLLM + n8n)
	Requires=docker.service
	After=docker.service network-online.target
	
	[Service]
	Type=oneshot
	RemainAfterExit=yes
	WorkingDirectory=/root/ai-stack
	ExecStart=/usr/bin/docker compose up -d
	ExecStop=/usr/bin/docker compose down
	TimeoutStartSec=120
	
	[Install]
	WantedBy=multi-user.target
	EOF
```
- systemctl daemon-reload
  systemctl enable --now ai-stack
  systemctl status ai-stack
  *R= Commands for make the service works*


## **Step 5 : Final Securing**

- We configure UFW to take charge of docker's new ports
	-    ufw allow 5678/tcp
		ufw allow 4000/tcp
		ufw allow 3000/tcp
		ufw enable
		ufw status verbose

- We check if any container has the power to undertake my OS
	- docker inspect $(docker ps -q) | grep -i docker.sock



## **END Step : What I Learned and I Doesn't Wrote Before**

- Ollama is 2 things : A "Play Store" of AI and the engine that executes it.
- The Importance of a GPU