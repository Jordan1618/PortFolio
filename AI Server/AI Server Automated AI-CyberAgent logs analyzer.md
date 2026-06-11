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
