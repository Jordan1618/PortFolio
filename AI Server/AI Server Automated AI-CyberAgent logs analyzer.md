## **Project Specifications : Vector + Loki + Grafana + n8n**

- First I had to check what's on the server :
	- hostnamectl
		lscpu
		free -h
		df -h
		docker ps -a
		docker network ls
		docker volume ls
		ss -tulpn

- Installation schema : 
	- [SRV-LNX-01..N]  ── Vector Agent ──┐
		[SRV-WIN-01..N]  ── Vector Agent ──┤
		                                    ▼
		                          [Vector Aggregator]
		                          (port 9000, container séparé ou hôte)
		                                    │
		                                    ▼
		                                 [Loki]
		                            (stockage + index)
		                                    │
		                            [Grafana] ──┘  (lit Loki + Prometheus)
		                                    │
		                                    ▼
		                    [n8n — Schedule toutes les 30min]
		                                    │
		                          ┌─────────┴──────────┐
		                          ▼                    ▼
		                    Requête Loki         (rien d'autre)
		                          │
		                          ▼
		                   Prompt Mistral
		                   "Analyse ces logs.
		                    Si critique, commence
		                    ta réponse par CRITIQUE:"
		                          │
		                          ▼
		                  Parse réponse Mistral
		                          │
		                  ┌───────┴────────┐
		                  │                │
		            commence par      commence par
		            "CRITIQUE:"       autre chose
		                  │                │
		                  ▼                ▼
		           Mail immédiat     Attendre le prochain
			           objet:            cycle de 30min
		           "🚨 CRITIQUE: X"  (n8n ne fait rien,
		                            le scheduler reviendra)
		                  │                │
		                  └───────┬────────┘
		                          ▼
		                       SMTP SITIV
		                  → X.X@X.com

## **Loki Installation :**

- Loki is a log storage with a search system crated by Grafana.
- Loki is a 3 parts system Loki as the brain that stores and serarchs, Promtail as the collector that reads and sends to Loki and Grafana as the dashboard.
- We need to configure the loki-config file :
	- cat << EOF > loki-config.yml
		auth_enabled: false
		
		server:
		  http_listen_port: 3100
		
		common:
		  path_prefix: /loki
		  storage:
		    filesystem:
		      chunks_directory: /loki/chunks
		      rules_directory: /loki/rules
		  replication_factor: 1
		  ring:
		    kvstore:
		      store: inmemory
		
		schema_config:
		  configs:
		    - from: 2025-01-01
		      store: tsdb
		      object_store: filesystem
		      schema: v13
		      index:
		        prefix: index_
		        period: 24h
		
		storage_config:
		  filesystem:
		    directory: /loki/chunks
		
		limits_config:
		  retention_period: 90d
		
		ruler:
		  storage:
		    type: local
		    local:
		      directory: /loki/rules
		  rule_path: /loki/rules
		EOF

- After we configure the docker-config file and create a date-file for loki's storage:
	- cat << EOF > docker-compose.yml
		services:
		  loki:
		    image: grafana/loki:latest
		    container_name: loki
		
		    restart: unless-stopped
		
		    ports:
		      - "3100:3100"
		
		    command:
		      - "-config.file=/etc/loki/local-config.yaml"
		
		    volumes:
		      - ./loki-config.yml:/etc/loki/local-config.yaml
		      - ./data:/loki
		EOF
	- mkdir -p data && chmod -R 777 data

- After the usual docker command :
	- docker compose up -d && docker ps

- The clean command after each change in docker files : 
	- docker compose down && rm -rf data/* && mkdir -p data/chunks data/rules && docker compose up -d

## **Loki Quick Notes (what actually broke my setup) :**

- I didn't have Consul installed so the connection was refused and Loki kept crashing in a loop
- Concept to master : A RIng, Scheduler, Consul/KV Store, TSDB, Fs storage, Standalone mode
	1) The ring = the coordination system
	2) Scheduler = system that distributes queries
	3) Consul/KV Store = Stores shared data
	4) TSDB (=Time Series Database Format)= Index + Stored
	5) FsStorage = Logs saved on local disk
	6) That make everything works

## **Connect n8n and Loki + Caddy on n8n to have secure cookies:**

- Command to check the state of the network, then connect if necessary: 
	- docker network ls
	- docker network inspect ai-internal
	- docker network ls --format "{{.Name}}" | grep -E "monitor|loki"
	- After :
	- docker network connect loki_default n8n
	- docker inspect n8n --format '{{range $k, $v := .NetworkSettings.Networks}}{{$k}} {{end}}'
	- docker exec n8n wget -qO- http://loki:3100/ready

- Changes the n8n file to make caddy works and create a docker backup in case loki crashs :
	- cd /root/ai-stack && \
		cp -f docker-compose.yml docker-compose.yml.bak && \
		cat > docker-compose.yml <<'EOF'
		services:
		
		  ollama:
		    image: ollama/ollama:latest
		    container_name: ollama
		    restart: unless-stopped
		    environment:
		      - OLLAMA_HOST=0.0.0.0:11434
		      - OLLAMA_KEEP_ALIVE=-1
		    networks:
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
		      - N8N_PROXY_HOPS=1
		      - WEBHOOK_URL=https://X.X.X.X:8443/
		    networks:
		      - ai-internal
		      - loki_default
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
		
		  loki_default:
		    external: true
		    name: loki_default
		
		volumes:
		  ollama_data:
		  webui_data:
		  n8n_data:
		EOF

- We activate and check :
	- docker compose up -d n8n
	- docker inspect n8n --format '{{range $k, $v := .NetworkSettings.Networks}}{{$k}} {{end}}'
		- Check n8n network after restart
	- docker exec n8n wget -qO- http://loki:3100/ready
		- Retry loki by n8n

- Installation of Caddy :
	- apt-get install -y debian-keyring debian-archive-keyring curl
		- debian-keyring stands for debian packet signature and debian-archive-keyring is idem but for 3 more GPG key sources, there are very usefull for apt
	- curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \ | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
		- curl download, -1sLf are for force TLS v1 in silent with HTTP redirection and prints fail if happens
		- | sends to gpg, gpg --dearmor converts text format to binary format, -o is for the destination repo, it stands for origin.
	- curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \ | sudo tee /etc/apt/sources.list.d/caddy-stable.list
		- Difference between tee and cat << EOF> 'EOF' : Tee writes in a doc and prints to screen while cat + EOF only writes in a doc. Tee is use by a sudo + | , when cat + EOF for a conf/script file
	- Why Caddy ? Because no let's encrypt needed and clean HTTPS automatically by one only access port. 
	- apt-get update && sudo apt-get install -y caddy

- Configuration of Caddy :
	- sudo tee /etc/caddy/Caddyfile << 'EOF'
		https://192.168.X.X:8443 {
		    tls internal
		
		    reverse_proxy localhost:5678 {
		        header_up X-Forwarded-Proto https
		        header_up Host {host}
		    }
		}
		
		https://192.168.X.X:8444 {
		    tls internal
		
		    reverse_proxy localhost:3000 {
		        header_up X-Forwarded-Proto https
		        header_up Host {host}
		    }
		}
		EOF
	- systemctl enable caddy
	- systemctl restart caddy
	- systemctl status caddy