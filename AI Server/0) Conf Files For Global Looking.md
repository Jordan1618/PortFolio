
# AI Infrastructure & Monitoring Stack

## **1. Configuration File Inventory**

### **AI Stack**

- `/root/ai-stack/docker-compose.yml`
- `/root/ai-stack/litellm_config.yaml`

### **Monitoring Stack**

- `/root/monitoring/docker-compose.yml`
- `/root/monitoring/prometheus.yml`
- `/root/monitoring/netdata.conf`

### **Logging & Routing Stack**

- `/root/monitoring/loki/docker-compose.yml`
- `/root/monitoring/loki/loki-config.yml`
- `/root/vector/docker-compose.yml`
- `/root/vector/config/aggregator.toml`

---

## **2. Full Configuration Files**

### **/root/ai-stack/docker-compose.yml**

```yaml
services:
  ollama:
    image: ollama/ollama
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
    image: ghcr.io/berriai/litellm
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
    image: ghcr.io/open-webui/open-webui
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
    image: n8nio/n8n
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
      - WEBHOOK_URL=[https://10.0.1.180:8443/](https://10.0.1.180:8443/)
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
````

### **/root/ai-stack/litellm_config.yaml**

YAML

```
model_list:
  - model_name: mistral-large
    litellm_params:
      model: ollama/mixtral:8x7b
      api_base: http://ollama:11434

litellm_settings:
  num_retries: 3
  request_timeout: 600
  telemetry: false
```

### **/root/monitoring/docker-compose.yml**

YAML

```
services:
  node-exporter:
    image: prom/node-exporter.8.1
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc
      - /sys:/host/sys
      - /:/rootfs
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
    network_mode: host

  prometheus:
    image: prom/prometheus.45.0
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    network_mode: host

  grafana:
    image: grafana/grafana:10.2.0
    container_name: grafana
    restart: unless-stopped
    environment:
      - GF_SERVER_HTTP_PORT=3001
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana
    network_mode: host

volumes:
  grafana-storage:
```

### **/root/monitoring/prometheus.yml**

YAML

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'serveur-ia-metrics'
    static_configs:
      - targets: ['127.0.0.1:9100']
```

### **/root/monitoring/netdata.conf**

Extrait de code

```
[web]
    bind to = 0.0.0.0
    allow connections from = localhost 127.0.0.1 172.16.0.0/12 192.168.0.0/16 10.0.0.0/8
    allow metrics from = *
```

### **/root/monitoring/loki/docker-compose.yml**

YAML

```
services:
  loki:
    image: grafana/loki
    container_name: loki
    restart: unless-stopped
    ports:
      - "3100:3100"
    command:
      - "-config.file=/etc/loki/local-config.yaml"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - ./data:/loki
```

### **/root/monitoring/loki/loki-config.yml**

YAML

```
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
```

### **/root/vector/docker-compose.yml**

YAML

```
services:
  vector:
    image: timberio/vector:0.38.0-alpine
    container_name: vector-aggregator
    restart: unless-stopped
    ports:
      # Réception logs agents — exposé sur toutes les interfaces LAN
      - "0.0.0.0:9000:9000"
      # Syslog UDP legacy
      - "0.0.0.0:9514:9514/udp"
      # Métriques Prometheus — loopback uniquement
      # Prometheus est en mode host → localhost suffit
      - "127.0.0.1:9598:9598"
    volumes:
      - /root/vector/config:/etc/vector:ro
    command: ["--config-toml", "/etc/vector/aggregator.toml"]
    networks:
      - loki_default
      # Vector a besoin de loki_default pour résoudre "loki"
      # et y envoyer les logs via HTTP
      # Pas besoin d'autres réseaux : Prometheus est en mode host
      # et scrape directement localhost:9598

networks:
  loki_default:
    external: true
    name: loki_default
```

### **/root/vector/config/aggregator.toml**

Ini, TOML

```
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

if !exists(.timestamp) {
.timestamp = now()
}

del(.agent)
del(.ecs)
del(.input)
del(.log)

if !exists(.site) { .site = "default" }
if !exists(.log_type) { .log_type = "generic" }
if !exists(.source_os) { .source_os = "unknow" }
'''

[sinks.loki_out]
type = "loki"
inputs = ["enrich"]
endpoint = "http://loki:3100"
encoding.codec = "json"

labels = { job = "vector", hostname = "{{ hostname }}", level = "{{ level }}", log_type = "{{ log_type }}", os = "{{ source_os }}", site = "{{site}}" }

batch.max_bytes = 1048576
batch.timeout_secs = 5
request.retry_attempts = 3
request.retry_initial_backoff_secs = 1

[sinks.prometheus_exporter]
type = "prometheus_exporter"
inputs = ["internal_metrics"]
address = "0.0.0.0:9598"
```

## **3. Related & Orphan Runtime Information**

### **Global Network Topology**

- **ai-internal**
    
    - **Driver**: `bridge`
        
    - **Scope**: `local`
        
    - **Type**: External
        
- **loki_default**
    
    - **Driver**: `bridge`
        
    - **Scope**: `local`
        
    - **Type**: External
        
- **host**
    
    - **Driver**: `host`
        
    - **Scope**: `local`
        
    - **Type**: Native System Network Mode