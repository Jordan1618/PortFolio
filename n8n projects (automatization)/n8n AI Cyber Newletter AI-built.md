# Project Overview: Self-Hosted & Zero-Cost Automation Pipeline

## ## 1. Executive Summary

This project aims to deploy a robust, **100% self-hosted, and zero-cost automation workflow inside a local VM.. The system automatically collects data from various public APIs, processes and synthesizes the information using an Artificial Intelligence (LLM - Mistral 7B in our case), and distributes an executive summary via a secure email channel.

The entire architecture is designed with two non-negotiable constraints:

- **Absolute Port Isolation:** No ports are exposed to the outside world to ensure maximum security.
- **Zero-Budget Execution:** Every component used is free and/or open-source.
    

## ## 2. Core Technical Stack

The infrastructure relies on containerized services running on a local Docker network, completely isolated from hostile external traffic.

|**Component**|**Technology**|**Role & Utility**|**Cost**|
|---|---|---|---|
|**Orchestrator**|`n8n (Self-Hosted)`|Workflow execution, scheduling (Cron), and API management.|**0.00 €**|
|**AI Engine**|`Ollama` OR `OpenRouter`|**Ollama:** Local inference (e.g., Mistral/Llama) for data sovereignty.<br><br>  <br><br>**OpenRouter:** Free cloud tier to save local RAM.|**0.00 €**|
|**Mail Gateway**|`Brevo SMTP Relay`|Secure transactional email delivery (up to 300 emails/day).|**0.00 €**|
|**Security**|`UFW + Docker Network`|System-level packet filtering and isolated inter-container network.|**0.00 €**|
|**Storage**|`SQLite`|Lightweight, embedded database file for n8n execution data.|**0.00 €**|

## ## 3. Operational Workflow Logic

The automation script runs sequentially through 4  phases:

```
 [Cron Trigger] ──> [Fetch Public APIs] ──> [Data Cleaning (JS)] ──> [LLM Synthesis] ──> [SMTP Email Egress]
```

1. **Triggering:** An internal Cron node wakes up the workflow at scheduled intervals (each Monday/Wednesday/Friday at 6am so i can read news in public transports and one time a week for a global cyber one to ).
    
2. **Ingestion:** Asynchronous HTTP nodes pull raw JSON data from public APIs (Weather, News, Crypto, etc.).
    
3. **Sanitization:** A code block normalizes the data, filters unnecessary metadata, and creates a clean prompt.
    
4. **AI Synthesis & Delivery:** The LLM summarizes the dataset into a concise markdown text, which is then wrapped in an email template and sent via Brevo (Port 587 TLS).
    

## ## 4. Security & Hardening Strategy

### ### Network Isolation

To block any external intrusion vectors, the firewall rules are strictly hardened:

- **Inbound Traffic:** Blocked by default (`ufw default deny incoming`).
    
- **n8n Binding:** Hard-bound to the local loopback adapter (`127.0.0.1:5678`), meaning the dashboard is completely invisible to the outside internet.
    
- **Inter-container Communication:** Services talk to each other via an internal Docker bridge network (`internal_secure_net`). No internal backend ports (like Ollama's `11434`) are mapped to the host OS.
    

### ### Secrets Management

All sensitive variables (`N8N_ENCRYPTION_KEY`, API tokens, SMTP credentials) are managed through a restricted `.env` file (`chmod 600`) and are never hardcoded into the configuration files.