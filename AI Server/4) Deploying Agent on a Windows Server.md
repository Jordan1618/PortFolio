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
		- The purpose is to get the Agent's Packets and zip-dezip la conf sous C:\Vector
	- New-Item -ItemType Directory -Path "C:\vector\config" -Force ; New-Item -ItemType Directory -Path "C:\vector\logs" -Force
		- We created two directories with the path and the execution now
		- I'd remarked PowerSheel always add a command to add a type like -ItemType for Directory or -Path for "C:\vector\logs". I think nothing is declared, each property has to be set up with an option.
	- Move-Item "C:\vector\bin\vector.exe" "C:\vector\vector.exe" -Force
	- C:\vector\vector.exe --version
	- Get-ChildItem C:\vector -Recurse -Filter "vector.exe"
		- This command helps to search for a vector.exe in case it wasn't with previous commands.
	- $config = @'
		data_dir = "C:\\vector\\logs"
		
		[sources.win_security]
		type = "windows_event_log"
		channels = ["Security"]
		
		[sources.win_system]
		type = "windows_event_log"
		channels = ["System"]
		
		[sources.win_application]
		type = "windows_event_log"
		channels = ["Application"]
		
		[sources.win_updates]
		type = "windows_event_log"
		channels = ["Microsoft-Windows-WindowsUpdateClient/Operational"]
		
		[sources.inventory_http]
		type = "http_server"
		address = "127.0.0.1:9001"
		decoding.codec = "json"
		
		[transforms.enrich_windows]
		type = "remap"
		inputs = ["win_security", "win_system", "win_application", "win_updates"]
		source = '''
		.hostname = get_hostname!()
		.source_os = "windows"
		.site = "saint-chamond"
		
		.log_type = if exists(.channel) {
		  ch = downcase(to_string!(.channel))
		  if contains(ch, "security") { "security" }
		  else if contains(ch, "system") { "system" }
		  else if contains(ch, "application") { "application" }
		  else if contains(ch, "windowsupdate") { "windows_update" }
		  else { "windows_generic" }
		} else { "windows_generic" }
		
		level_raw = downcase(to_string(.level) ?? "info")
		.level = if includes(["critical", "error"], level_raw) { "error" }
		  else if includes(["warning", "warn"], level_raw) { "warning" }
		  else { "info" }
		
		eid = to_int(.system.event_id.value) ?? 0
		.event_id = to_string(eid)
		.level = if includes([4625, 4740, 1102, 4648, 4719, 4964, 1074, 6008], eid) { "critical" }
		  else { .level }
		
		.message = to_string(.message) ?? to_string(.rendered_message) ?? "no message"
		if !exists(.timestamp) { .timestamp = now() }
		'''
		
		[transforms.enrich_inventory]
		type = "remap"
		inputs = ["inventory_http"]
		source = '''
		.hostname = get_hostname!()
		.source_os = "windows"
		.site = "saint-chamond"
		if !exists(.timestamp) { .timestamp = now() }
		if !exists(.level) { .level = "info" }
		if !exists(.message) { .message = "inventaire" }
		'''
		
		[transforms.filter_noise]
		type = "filter"
		inputs = ["enrich_windows"]
		condition = 'includes(["warning", "error", "critical"], .level)'
		
		[sinks.to_aggregator_logs]
		type = "http"
		inputs = ["filter_noise"]
		uri = "http://10.0.1.180:9000"
		encoding.codec = "json"
		batch.max_bytes = 1048576
		batch.timeout_secs = 5
		request.retry_attempts = 5
		
		[sinks.to_aggregator_inventory]
		type = "http"
		inputs = ["enrich_inventory"]
		uri = "http://10.0.1.180:9000"
		encoding.codec = "json"
		batch.max_bytes = 2097152
		batch.timeout_secs = 10
		request.retry_attempts = 5
		'@
		[System.IO.File]::WriteAllText("C:\vector\config\agent.toml", $config, (New-Object System.Text.UTF8Encoding($false)))

		- GLOBAL EXPLAINATION : Everything has been told in my natural langage. But to resume : First a Metadata collect, Second a Log-type classification with a basic and a critical filters, Third a level hierarchy (no need to define because Vector, before each entry, has already converts JSON and we have only filtered one type, but level is another one we just called), Forth an Extraction and EID Treatment, Fifth a Critical-security-filter in case level triggers, Sixth it keeps the explicative message and put a timestamp or keeps the original, Seventh it takes data from inventory_http (we will define later), Eighth a anti-noise filter and Nineth the Both sinks to drop right data in right place.

		- BUT AFTER MULTIPLE ATTEMPTS, IT FAILED CONSTANTLY, So I went on Vector website to find the right command : https://vector.dev/docs/setup/installation/package-managers/msi/ that gave me : powershell Invoke-WebRequest https://packages.timber.io/vector/0.56.0/vector-x64.msi -OutFile vector-0.56.0-x64.msi msiexec /i vector-0.56.0-x64.msi
		
	- powershell Invoke-WebRequest https://packages.timber.io/vector/0.56.0/vector-x64.msi -OutFile vector-0.56.0-x64.msi>> msiexec /i vector-0.56.0-x64.msi
		- Installation 0.56.0
	- Copy-Item "C:\vector\bin\vector.exe" "C:\vector\vector.exe" -Force
		- Binary root updated
	- $config = @'
		data_dir = "C:\\vector\\logs"
		
		[sources.win_security]
		type = "windows_event_log"
		channels = ["Security"]
		
		[sources.win_system]
		type = "windows_event_log"
		channels = ["System"]
		
		[sources.win_application]
		type = "windows_event_log"
		channels = ["Application"]
		
		[sources.win_updates]
		type = "windows_event_log"
		channels = ["Microsoft-Windows-WindowsUpdateClient/Operational"]
		
		[sources.inventory_http]
		type = "http_server"
		address = "127.0.0.1:9001"
		decoding.codec = "json"
		
		[transforms.enrich_windows]
		type = "remap"
		inputs = ["win_security", "win_system", "win_application", "win_updates"]
		source = '''
		.hostname = get_hostname!()
		.source_os = "windows"
		.site = "saint-chamond"
		
		.log_type = if exists(.channel) {
		  ch = downcase(to_string!(.channel))
		  if contains(ch, "security") { "security" }
		  else if contains(ch, "system") { "system" }
		  else if contains(ch, "application") { "application" }
		  else if contains(ch, "windowsupdate") { "windows_update" }
		  else { "windows_generic" }
		} else { "windows_generic" }
		
		level_raw = downcase(to_string(.level) ?? "info")
		.level = if includes(["critical", "error"], level_raw) { "error" }
		  else if includes(["warning", "warn"], level_raw) { "warning" }
		  else { "info" }
		
		eid = to_int(.system.event_id.value) ?? 0
		.event_id = to_string(eid)
		.level = if includes([4625, 4740, 1102, 4648, 4719, 4964, 1074, 6008], eid) { "critical" }
		  else { .level }
		
		.message = to_string(.message) ?? to_string(.rendered_message) ?? "no message"
		if !exists(.timestamp) { .timestamp = now() }
		'''
		
		[transforms.enrich_inventory]
		type = "remap"
		inputs = ["inventory_http"]
		source = '''
		.hostname = get_hostname!()
		.source_os = "windows"
		.site = "saint-chamond"
		if !exists(.timestamp) { .timestamp = now() }
		if !exists(.level) { .level = "info" }
		if !exists(.message) { .message = "inventaire" }
		'''
		
		[transforms.filter_noise]
		type = "filter"
		inputs = ["enrich_windows"]
		condition = 'includes(["warning", "error", "critical"], .level)'
		
		[sinks.to_aggregator_logs]
		type = "http"
		inputs = ["filter_noise"]
		uri = "http://10.0.1.180:9000"
		encoding.codec = "json"
		batch.max_bytes = 1048576
		batch.timeout_secs = 5
		request.retry_attempts = 5
		
		[sinks.to_aggregator_inventory]
		type = "http"
		inputs = ["enrich_inventory"]
		uri = "http://10.0.1.180:9000"
		encoding.codec = "json"
		batch.max_bytes = 2097152
		batch.timeout_secs = 10
		request.retry_attempts = 5
		'@
		[System.IO.File]::WriteAllText("C:\vector\config\agent.toml", $config, (New-Object System.Text.UTF8Encoding($false)))
		- The right conf file to copy paste
	- C:\vector\vector.exe validate --config-toml C:\vector\config\agent.toml
		- To check
	- C:\vector\vector.exe service install --config C:\vector\config\agent.toml ; sc.exe config vector obj= "LocalSystem" ; Start-Service vector
		- Create Vector as a Service Windows
	- Get-Service -Name vector | Select-Object Name, Status, StartType
		- Running and Starting checks
	- Set-Service -Name vector -StartupType Automatic ; Get-Service -Name vector | Select-Object Name, Status, StartType
- OR 
	- I Can use this one : C:\vector\vector.exe service install --config C:\vector\config\agent.toml ; sc.exe config vector obj= "LocalSystem" start= auto ; Start-Service vector ; Get-Service -Name vector | Select-Object Name, Status, StartType
- NOW we will make the inventory script :
	- New-Item -ItemType Directory -Path "C:\vector\scripts" -Force
	- .
	- ......................