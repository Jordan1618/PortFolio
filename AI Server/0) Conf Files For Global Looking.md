# AI Infrastructure & Monitoring Stack

## **1. Configuration File Inventory**

### **AI Stack**

- `/root/ai-stack/docker-compose.yml`
- `/root/ai-stack/litellm_config.yaml`

### **Monitoring Stack**

- `/root/monitoring/docker-compose.yml`
- `/root/monitoring/prometheus.yml`
- `/root/monitoring/netdata.conf`

### **Logging Stack**

- `/root/monitoring/loki/docker-compose.yml`
- `/root/monitoring/loki/loki-config.yml`

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
  
```
