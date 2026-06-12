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

		- Explainations : tls internal stands for internal Caddy's SSL Certificate; reverse_proxy to activate the redirection to other destination points; localhost because docker exposes n8n and others directly; headers means for X_Forwarded_Proto users are on HTTPS and for Host to keep the right IP/Domain.
		- TLS means Transport Layer Security and acts as a digital identity licence
	- systemctl enable caddy
	- systemctl restart caddy
	- systemctl status caddy

## **Setup Vector Aggregator (Agent and Central Server) :**

- First we are looking for each docker on central server (ai server), on which network they are 
	- docker ps -q | xargs docker inspect --format '{{.Name}} → {{range $k, $v := .NetworkSettings.Networks}}{{$k}} {{end}}'
		- docker ps -q gives IDs of containers; xargs transfers to the foloowing command; docker inspect --format tells "For each container give me the name and the network"; {{.NAME}} means the current (.) object name (NAME) on docker ( {{ }} ); range stands for "for i in range"; $k & $v are two variables inside the range; := means "filled with found values"; .NetworkSettings & .Network signifies the right property to take. And End to end the '' space.
		- docker has a syntax close to the GO langage so that's why it's unusual.
- After we chose to create him its container docker:
	- mkdir -p /root/vector/config
- Now we are configurating its files :
	- cat > /root/vector/config/aggregator.toml << 'EOF'  
		[sources.http_agents]  
		type = "http_server"  
		address = "0.0.0.0:9000"  
		encoding.codec = "json"
		
		[sources.syslog_legacy]  
		type = "syslog"  
		address = "0.0.0.0:9514"  
		mode = "udp"
		
		[sources.internal_metrics]  
		type = "internal_metrics"
		
		[transforms.enrich]  
		type = "remap"  
		inputs = ["http_agents", "syslog_legacy"]  
		source = '''  
		if !exists(.hostname) {  
		.hostname = string(.host) ?? "unknown"  
		}
		
		level_raw = downcase(string(.level ?? .severity ?? "info"))  
		.level = if includes(["critical","crit","emerg","alert","fatal"], level_raw) {  
		"critical"  
		} else if includes(["error","err"], level_raw) {  
		"error"  
		} else if includes(["warning","warn"], level_raw) {  
		"warning"  
		} else {  
		"info"  
		}
		
		if exists(.MESSAGE) {  
		.message = string!(.MESSAGE)  
		del(.MESSAGE)  
		}
		
		if !exists(.timestamp) {  
		.timestamp = now()  
		}
		
		del(.agent)  
		del(.ecs)  
		del(.input)  
		del(.log)  
		'''
		
		[sinks.loki_out]  
		type = "loki"  
		inputs = ["enrich"]  
		endpoint = "[http://loki:3100](http://loki:3100/)"  
		encoding.codec = "json"
		
		labels = {  
		"job" = "vector",  
		"hostname" = "{{ hostname }}",  
		"level" = "{{ level }}",  
		"log_type" = "{{ log_type }}",  
		"os" = "{{ source_os }}",  
		"site" = "default"  
		}
		
		batch.max_bytes = 1048576  
		batch.timeout_secs = 5  
		request.retry_attempts = 3  
		request.retry_initial_backoff_secs = 1
		
		[sinks.prometheus_exporter]  
		type = "prometheus_exporter"  
		inputs = ["internal_metrics"]  
		address = "0.0.0.0:9598"  
		EOF
	- To put it in a nutshell, it helps to convert Linux and Windows logs into a unique format, readable by loki.

- After we make in form the Vector container :
	- cat > /root/vector/docker-compose.yml << 'EOF'
		services:
		  vector:
		    image: timberio/vector:0.38.0-alpine
		    container_name: vector-aggregator
		    restart: unless-stopped
		    ports:
		      - "0.0.0.0:9000:9000"
		      - "0.0.0.0:9514:9514/udp"
		      - "127.0.0.1:9598:9598"
		    volumes:
		      - /root/vector/config:/etc/vector:ro
		    command: ["--config", "/etc/vector/aggregator.toml"]
		    networks:
		      - loki_default
		
		networks:
		  loki_default:
		    external: true
		    name: loki_default
		EOF
	- This configuration speaks about the network and the container itself. The container has its image, name, restart option, listening ports, storage inside the container. Both networks leads to internal (docker) or external (the current server). Command stands for when starting, add more instructions.

- Next, we test the vector to check if each green flag is raised :
	- cd /root/vector
	- sed -i 's|"--config", "/etc/vector/aggregator.toml"|"--config-toml", "/etc/vector/aggregator.toml"|g' /root/vector/docker-compose.yml
		- It's the necessary upgrade because the Parse tool, doesn't manage multiple columns, it prefers one long line.
	- sed -i 's/encoding\.codec = "json"/decoding.codec = "json"/' /root/vector/config/aggregator.toml
	- sed -i '52s/decoding\.codec/encoding.codec/' /root/vector/config/aggregator.toml
	- curl -s -X POST http://localhost:9000 \
	  -H 'Content-Type: application/json' \
	  -d '{"hostname":"srv-test","source_os":"linux","log_type":"syslog","level":"warning","message":"TEST PIPELINE OK"}' \
		&& sleep 6 \
		&& curl -s "http://localhost:3100/loki/api/v1/query_range?query=%7Bhostname%3D%22srv-test%22%7D&limit=5&start=$(date -d '5 minutes ago' +%s)000000000&end=$(date +%s)000000000" \
		| python3 -c "
		import sys, json
		data = json.load(sys.stdin)
		results = data.get('data', {}).get('result', [])
		if not results:
		    print('ECHEC : no log in Loki')
		else:
		    for stream in results:
		        for ts, msg in stream.get('values', []):
		            log = json.loads(msg)
		            print(f'OK : [{log.get(\"level\",\"?\").upper()}] [{log.get(\"hostname\",\"?\")}] {log.get(\"message\",\"?\")}')
		"
		- explainations : Curl -s -X POST means to Http data sent to vector; H is for making vector understands the following data as json. The second curl is for Loki

## Setup Vector Agent (Part 2) :

- First 


## **Configure n8n and Mistral :**

- For beginning, I go on my IP by HTTPS and port 8443, opened before and I log with my IDs. 
- After I create a workflow, add a schedule trigger every 30minutes, after I add an HTTP Request node with 4 parameters :
	- start / {{ Math.floor((Date.now() - 30 * 60 * 1000) / 1000) }}000000000
	- end / {{ Math.floor(Date.now() / 1000) }}000000000
	- query / {level=~"warning|error|critical"}
	- limit / 200
	- The method is GET and http://loki:3100/loki/api/v1/query_range
- Therefore, I put a Code in JS node named `Parser Loki + Prompt`. This node aggregates data streams, cleans Vector's JSON serialization string lines, and builds the strict engineering contextual cybersecurity prompt.
	- // Extraction de la réponse Loki
		const items = $input.all();
		let extractedLogs = [];
		
		if (items[0] && items[0].json && items[0].json.data && items[0].json.data.result) {
		  const results = items[0].json.data.result;
		  
		  for (const streamObj of results) {
		    const labels = streamObj.stream || {};
		    for (const valueArray of streamObj.values) {
		      // valueArray[0] = timestamp nanoseconde, valueArray[1] = message brut
		      const rawLine = valueArray[1];
		      let message = rawLine;
		      let level = labels.level || "UNKNOWN";
		      let site = labels.site || "default";
		      let hostname = labels.hostname || "unknown";
		      let logType = labels.log_type || "generic";
		      let os = labels.os || "linux";
		      
		      // Désérialisation du JSON envoyé par votre Vector Aggregator
		      try {
		        const parsed = JSON.parse(rawLine);
		        message = parsed.message || parsed.msg || rawLine;
		        if (parsed.level) level = parsed.level;
		        if (parsed.site) site = parsed.site;
		        if (parsed.hostname) hostname = parsed.hostname;
		        if (parsed.log_type) logType = parsed.log_type;
		        if (parsed.source_os) os = parsed.source_os;
		      } catch (e) {
		        // Pas du JSON, on conserve la ligne brute
		      }
		      
		      extractedLogs.push({
		        timestamp: new Date(parseInt(valueArray[0].substring(0, 13))).toISOString(),
		        site: site,
		        hostname: hostname,
		        log_type: logType,
		        os: os,
		        level: level.toUpperCase(),
		        message: message
		      });
		    }
		  }
		}
		
		// Validation : si aucun log n'est remonté, on lève l'indicateur d'arrêt
		if (extractedLogs.length === 0) {
		  return [{ json: { skip: true, prompt: "", total_logs: 0, stats: {} } }];
		}
		
		// Tri strict par criticité décroissante (CRITICAL -> ERROR -> WARNING)
		const priorityScore = { 'CRITICAL': 3, 'ERROR': 2, 'WARNING': 1, 'UNKNOWN': 0 };
		extractedLogs.sort((a, b) => (priorityScore[b.level] || 0) - (priorityScore[a.level] || 0));
		
		// Collecte des statistiques globales pour le contexte du prompt
		const totalLogs = extractedLogs.length;
		const criticalCount = extractedLogs.filter(l => l.level === 'CRITICAL').length;
		const errorCount = extractedLogs.filter(l => l.level === 'ERROR').length;
		const warningCount = extractedLogs.filter(l => l.level === 'WARNING').length;
		const distinctHosts = [...new Set(extractedLogs.map(l => l.hostname))].join(', ');
		
		// Limitation à 50 logs pour préserver la fenêtre de contexte de l'IA
		const limitedLogs = extractedLogs.slice(0, 50);
		
		// Formatage de la liste textuelle des logs
		let logLines = limitedLogs.map((l, i) => 
		  `[${i+1}] [${l.level}] [Hôte: ${l.hostname}] [OS: ${l.os} / Type: ${l.log_type}] ${l.timestamp}\n     ${l.message}`
		).join('\n');
		
		// Construction du prompt directif
		let prompt = `Tu es IRAS, un ingénieur expert en cybersécurité pour une collectivité territoriale française.
		Analyse le lot de logs système suivants récupérés au cours des 30 dernières minutes.
		
		Statistiques : ${totalLogs} logs détectés | ${criticalCount} critiques | ${errorCount} erreurs | ${warningCount} alertes.
		Équipements concernés : ${distinctHosts}
		
		Logs à analyser :
		${logLines}
		
		CONSIGNES STRICTES :
		- Si tu détectes une menace active (intrusion, brute force, compromission, ransomware) ou une panne système paralysante : commence obligatoirement ta réponse par la mention exacte "CRITIQUE: " suivie d'un résumé du problème sur la même ligne.
		- Sinon, commence obligatoirement ta réponse par la mention exacte "RAPPORT: " suivie d'un titre global sur la même ligne.
		
		Tu dois formater le reste de ta réponse en respectant rigoureusement la structure textuelle suivante :
		---
		SCORE: [Note globale de gravité de 1 à 10]
		NIVEAU: [CRITICAL, HIGH, MEDIUM, ou LOW]
		RESUME: [Résumé synthétique des événements en 2 ou 3 phrases]
		FINDINGS:
		- [Premier fait marquant identifié]
		- [Second fait marquant identifié]
		ACTIONS:
		- [Action corrective immédiate conseillée]
		- [Mesure préventive conseillée]`;
		
		return [{ 
		  json: { 
		    skip: false, 
		    prompt: prompt, 
		    total_logs: totalLogs,
		    stats: {
		      total: totalLogs,
		      critical: criticalCount,
		      error: errorCount,
		      warning: warningCount,
		      hosts: distinctHosts
		    }
		  } 
		}];
		- Following the prompt creation, an **IF Node** named `Logs présents ?` handles the filtering path. It uses a **Boolean** type validation checking if `={{ $json.skip }}` is equal to `true`.
			- **Branch TRUE** leads to a `No Operation` node because no relevant logs occurred during the time frame.
			- **Branch FALSE** implies logs are found, driving the workflow down to our localized LLM.
		- Next, the HTTP Request node named `Analyse MIstral` posts the payload data to Ollama.
			- **URL** : `http://ollama:11434/api/generate`
			- **Method** : POST
			- **Body Content Type** : JSON
			- **Why `mistral:latest` ?** We switched from Mixtral (26GB) to Mistral 7B (4.1GB) because running inference on dual Intel Xeon CPUs without a GPU caused heavy memory exhaustion, resulting in connection drops. The 7B model runs 10x faster and is perfectly stable.
			- **Crucial formatting note** : The JSON Body field MUST be switched to **Expression mode (fx)** using `{{ JSON.stringify($json.prompt) }}` without wrapping quotes, ensuring string carriage returns don't break strict JSON syntax rules. We also append a custom 2-minute timeout option (`120000`).
		- Then, another JavaScript Code Node named `Parser Réponse IA` intercepts the raw text block output from Ollama (`$json.response`). It reads line-by-line using regular expressions to capture the scoring metrics, security findings, and remediation actions. It dynamically formats the data payload and cooks a custom subject line based on whether the AI issued a `CRITIQUE:` or `RAPPORT:` header flag.
		- Finally, the output splits into two downstream targets:
			1) `Print in n8n`: A debugging Code node displaying a clear, tabular view of the automated report within the n8n execution interface.
			2) `Send an Email`: An SMTP node configured to send a clean, high-visibility HTML styled dashboard template tracking system anomalies straight to the mailbox, currently awaiting official  mail relay authorizations.

## **Manual Log Injection Framework (Testing the core) :**

- To validate the workflow end-to-end and force the IF conditions to process different alert thresholds, we execute raw API cURL entries from our central server command line to test our pipeline ingestion limits:

```bash
# Force a Critical Security Incident Scenario
curl -s -X POST http://localhost:9000 \
  -H 'Content-Type: application/json' \
  -d '{"hostname":"SRV-AD-01","source_os":"windows","log_type":"security","level":"critical","message":"Auditing Success: Account logged on after 45 failures. TargetUserName: Administrator. IpAddress: 185.220.101.42","event_id":"4624","site":"default"}'

# Force a Standard Warning Syslog Report Scenario
curl -s -X POST http://localhost:9000 \
  -H 'Content-Type: application/json' \
  -d '{"hostname":"srv-proxy-lnx","source_os":"linux","log_type":"syslog","level":"warning","message":"WARNING: /dev/mapper/ubuntu--vg is 92% full. Free space: 4.2GB.","site":"default"}'