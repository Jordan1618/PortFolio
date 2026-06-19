## **How Will I Proceed :**

- 1) Preparing Ubuntu
- 2) Windows : Install Vector Agent + Script PS1
- 3) Test if Loki get Windows's data
- 4) Modify n8n Workflow
- 5) Global Test

## **What Am I Aiming To ?**

- 1) Collecting Various OS Event Logs
- 2) Having a System Inventory
- 3) Analyzing in real time my network traffic
- 4) Predicting risks and make recommandations

## **1) Looking for Ubuntu**

- cd /root/vector && docker compose up -d
- docker ps | grep vector
- ss -tlnp | grep 9000
- curl -s -o /dev/null -w "HTTP %{http_code}\n" -X POST http://localhost:9000 -H "Content-Type: application/json" -d '{"test":"ping","level":"info","hostname":"test","source_os":"linux","site":"default","log_type":"test","message":"test"}'
- cat << 'EOF' > /root/vector/config/aggregator.toml
		[sources.http_agents]
		type = "http_server"
		address = "0.0.0.0:9000"
		decoding.codec = "json"
		
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
		
		level_raw = downcase(to_string(.level) ?? to_string(.severity) ?? "info")
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
		
		if !exists(.timestamp) { .timestamp = now() }
		if !exists(.site) { .site = "default" }
		if !exists(.log_type) { .log_type = "generic" }
		if !exists(.source_os) { .source_os = "unknown" }
		if !exists(.event_id) { .event_id = "0" }
		if !exists(.message) { .message = "no message" }
		
		del(.agent)
		del(.ecs)
		del(.input)
		del(.log)
		'''
		
		[sinks.loki_out]
		type = "loki"
		inputs = ["enrich"]
		endpoint = "http://loki:3100"
		encoding.codec = "json"
		
		labels = { job = "vector", hostname = "{{ hostname }}", level = "{{ level }}", log_type = "{{ log_type }}", os = "{{ source_os }}", site = "{{ site }}" }
		
		batch.max_bytes = 1048576
		batch.timeout_secs = 5
		request.retry_attempts = 3
		request.retry_initial_backoff_secs = 1
		
		[sinks.prometheus_exporter]
		type = "prometheus_exporter"
		inputs = ["internal_metrics"]
		address = "0.0.0.0:9598"
		EOF
- docker restart vector-aggregator
- docker logs vector-aggregator --tail 20 2>&1 | grep -E "error|Error|started|listening"

Before any problem, I copied each config file (docker/config/systemd/cron) in [Infrastructure Config and Backup Files](Infrastructure%20Config%20and%20Backup%20Files.md).

## **2) Agent Deployment on a WS **

- First :
	- Invoke-WebRequest -Uri "https://packages.timber.io/vector/0.38.0/vector-0.38.0-x86_64-pc-windows-msvc.zip" -OutFile "$env:TEMP\vector.zip" ; Expand-Archive "$env:TEMP\vector.zip" -DestinationPath "C:\vector" -Force
	- The pru