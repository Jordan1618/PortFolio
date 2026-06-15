## **Why I Wanted To Explain More ?**

- After reading again my before paper [AI Server Automated AI-CyberAgent logs analyzer](AI%20Server%20Automated%20AI-CyberAgent%20logs%20analyzer.md) I realized it's confusing for everyone reading it. I want to make a better explanation and go deeper into each node.
- Now you can see the Pre-Final Version. In the future each node will be upgraded. I want to add more logs to the final analyse, to have an "instant" mod for critical log, to upgrade some prompts and vector filters.

![](Pasted%20image%2020260615112121.png)

## **The First Node : Schedule Trigger**

- The easiest one, you just have to chose your interval. In my case : 15 minutes.

## **The Second Node : Http GET + Loki **


 ![](Pasted%20image%2020260615113653.png)

- The GET + URL is a request to the API of Loki to get the data and its range to avoid saturating the server.
- The query and its "level=~" are used to target error levels named Warning/Error or Critical.
- The start and the end module are there for pointing the last 15 minutes to analyze. 
- The limit stands to protect the system from a logs tsunami attack.
- Why the command line is : {{ Math.floor((Date.now() - 15 * 60 * 1000) / 1000) }}000000000
	- This took the current milliseconds count and translate it into seconds by Math.floor and the following operation and the 9 zeros is a syntax for loki that requests a nanoseconds timestamp.

### Now we go check inside the container of Loki

- But there was some problem : Writing blocked
	- 1) Granted UID Loki Property of files and wrx rights on parents
		- chown -R 10001:10001 /root/monitoring/loki
		- chmod +x /root /root/monitoring
	- 2) Restart the container 
		- docker restart cbe86061a19a

- I made a backup before editing the conf :
	- cp /root/monitoring/loki/loki-config.yml /root/monitoring/loki/loki-config.yml.bak
	- cat << 'EOF' > /root/monitoring/loki/loki-config.yml
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
		    - from: "2025-01-01"
		      store: tsdb
		      object_store: filesystem
		      schema: v13
		      index:
		        prefix: index_
		        period: 24h
		
		storage_config:
		  filesystem:
		    directory: /loki/chunks
		
		ruler:
		  storage:
		    type: local
		    local:
		      directory: /loki/rules
		  rule_path: /loki/rules
		
		limits_config:
		  retention_period: 336h
		
		compactor:
		  working_directory: /loki/compactor
		  compaction_interval: 10m
		  retention_enabled: true
		  delete_request_store: filesystem
		  retention_delete_delay: 2h
		  retention_delete_worker_count: 150
		EOF

- Now the conf is right with a 14days longs logs autocleaner :
	- chown -R 10001:10001 /root/monitoring/loki && docker restart cbe86061a19a
	- docker ps | grep loki
	- docker logs cbe86061a19a 2>&1 | grep -E "Loki started|compactor" | head -n 10
	- docker logs --tail 30 cbe86061a19a

- If everything is right, we can go to the next node.

## **The Third Node : Http GET + Loki **

- It's an ETL code : Extract, Transform, Load. And there is a prompt at the end for mistral.
- Extraction of Loki data :
		2: const items = $input.all();
		3: let extractedLogs = [];
	- const and let are the word use to define a constance of a variable
	- The first is for store all before logs and the second is to prepare the empty plate to collect usefull logs
- The Null-Check :
	 5: if (items[0] && items[0].json && items[0].json.data && items[0].json.data.result) {
	 6:   const results = items[0].json.data.result;
	- The First line is to check if there is something taken from loki or not.
	- The Second is for create a syntax shortcut
- Extraction of labels :
	 8:   for (const streamObj of results) {
	 9:     const labels = streamObj.stream || {}
	- The purpose there is to take each label from logs like OS/Computer name or ...
	- If there is no label, thus {}
- Raw messages :
	10:     for (const valueArray of streamObj.values) {
	12:       const rawLine = valueArray[1];
	- 
