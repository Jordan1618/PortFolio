## **Before entering the subject: The Stack and The Hardware

- One main priority for this project is to have a Free-Open Source Stack
- The stack planned is : LTS Docker + Ollama + API REST + Mistral 22B(Q6) + API Gateway + OpenWebUI Interface + n8n + LiteLLM
- On the hardware side it's : Dell Server with ESXI + 256GB RAM + 2 Intel Xeon 2.10Ghz + 8TB Storage
## **Step 1 : Configure the Server

- Seting up a Raid 5 Replication Storage
- Installing a Ubuntu Resolute Raccoon 26.04 OS in CLI with a Rufus's Bootable USB Key
- Configurating the *ssh key to my main computer with apt install openssh-server -y* + add a *right user with usermod -aG sudo NAME* +  *apt update && apt full-upgrade -y* + apt install *unattended-upgrades && dpkg-reconfigure --priority=low unattended-upgrades*
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
	*Inernal network*
