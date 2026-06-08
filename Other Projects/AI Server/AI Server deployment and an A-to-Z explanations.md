## **Before entering the subject: The Stack and The Hardware

- One main priority for this project is to have a Free-Open Source Stack
- The stack planned is : LTS Docker + Ollama + API REST + Mistral 22B(Q6) + API Gateway + OpenWebUI Interface + n8n + LiteLLM
- On the hardware side it's : Dell Server with ESXI + 256GB RAM + 2 Intel Xeon 2.10Ghz + 8TB Storage
## **Step 1 : Configure the Server

- Seting up a Raid 5 Replication Storage
- Installing a Ubuntu 26.04 OS in CLI with a Rufus's Bootable USB Key
- Configurating the *ssh key to my main computer with apt install openssh-server -y* + add a *right user with usermod -aG sudo NAME* +  *apt update && apt upgrade -y* + apt install *unattended-upgrades && dpkg-reconfigure --priority=low unattended-upgrades*
## **Step 2 : Configure Docker + systemd

- 
