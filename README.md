# 🌐 Jordan's CV Hub & Portfolio

> **Welcome to my local knowledge base. This repository tracks my learning journey, showcases production infrastructure deployments, custom AI automations, and side-projects built with a deep-understanding approach. I did all this ReadMe by Myself, I wanted to make it more personnal and very representative of who am i**

---
## 🎯 My Vision and My Skills

> My long-term professional objective is to help organizations to deploy their own AI on local systems for produce, understand or make decisions. I'm learning how to use this new technology and make it useful/legal (especially with the EU laws) and sovereign. Through this journey I'm learning and testing different protocols, mindsets, IT culture and more globally CyberSecurity features. 
> 
> To be honest, I think IT is a very big world where it's difficult to know our place or specialization so I use my curiosity to lead me where I can develop my own Skills, Knowledge and Relationships. That's why this PortFolio is "English-Full".
> 
> My main projects are : 
> **Intelligence technologic agent(Personalized News Collector and Aggregator)/Sovereign AI Deployment (Log analyzer on a Production Infrastructure) and various personal projects (KaramelIA/Self-Learning Documentations/Financial assets on-mesure tracking).**

---
## 📂 Project Directory & Portfolio Index

### 🛡️ Cybersecurity & Local AI Stack
* 1. **I Server Deployment & Hardening:** Architecture of a local AI server on a Dell Server (ESXi, 2x Intel Xeon, 256GB RAM). Includes Docker deployment, strict network isolation (`ai-internal`), and a 150GB RAM limit to avoid DoS or crash risks.
    
2. **Log Ingestion Pipeline:** Automatic log collection using Vector agents to normalize Windows Security Logs (EventID 4624) and Linux syslogs into a centralized Grafana Loki database.
    
3. **n8n Data Parsing:** Custom JavaScript nodes inside n8n (null-checks, text-slicing for LLM context windows, and JSON restructuring) to clean and format raw data before sending it to the local AI.
    
4. **Remote Telemetry:** Deployment of remote Vector sensors on production Windows servers for real-time security tracking.
    
5. **Infrastructure-as-Code Repo:** Centralized repository for my production configs: `docker-compose.yml` master files, VRL rules (`aggregator.toml`), and LiteLLM proxy settings (`litellm_config.yaml`).
    
### 📊 Financial Data & Market Analytics

- **Deterministic Core Math (No-LLM):** Pure JavaScript routines for precise portfolio auditing and asset cost-basis tracking to eliminate AI math mistakes.
    
- **Daily Ingestion (07:30 AM):** A daily cron job compiling market data into a clean report sent via secure SMTP for my morning read.
    
- **Local AI Macro Analytics:** Local LLM used only for NLP, news aggregation, and finding trends for weekly reports.
    
- **Corporate Governance:** Automated tracking of upcoming Annual General Meetings (AGM) and registered shares (_"au nominatif"_).
    

### 🧠 Self-Learning & Knowledge Base

- **Core Knowledge Vault:** The biggest part of my portfolio. A repository of my notes, research, and technical summaries on infrastructure, systems, and computer science.
    
- **Systems Scripting:** A collection of lightweight sysadmin tools, local batch scripts, and cron maintenance routines.
    
- **Windows OS Remediation Utility:** A shell script designed to bypass local UAC for standard users, modify specific HKLM registry keys, and automate mandatory updates at 8:00 PM.
    

### 🎨 KaramelIa: Automated Content System

- **Content Pipeline:** An autonomous multimedia content generation system driven by AI workflows.
    
- **Video Processing & Publishing:** Dev logs tracking video automation, DaVinci Resolve workflows, and n8n sequences to auto-publish with SEO, descriptions, and schedules on YouTube, TikTok, and Instagram.
    

### 🇷🇺 Language Learning

- **Russian Acquisition Framework:** A workspace to track my progress, vocabulary, and grammar targets from beginner to advanced intermediate (A1 to B2).
    

## 🛠️ Technical Skills & Tools Stack

- **Infrastructure:** VMware ESXi, Linux Ubuntu/Debian CLI, Windows Server
    
- **Containers & Admin:** Docker, Docker Compose, systemd, Windows Registry, GPO, Active Directory, RDP
    
- **Logs & Telemetry:** Vector (VRL), Grafana Loki
    
- **Monitoring:** Prometheus, NodeExporter, Grafana Dashboards
    
- **Network & Security:** UFW, Caddy Server (Reverse Proxy & TLS), Network Segregation, Firewalls, SIEM, EDR/NDR
    
- **Automation:** n8n, JavaScript, ETL Pipelines
    
- **Local AI:** Ollama (Mistral 7B, Mixtral), LiteLLM Proxy
    

## 🎯 Strategic Principles

- **Absolute Data Sovereignty:** Zero logs, tokens, or network data ever leave the local network. Everything is 100% local and airtight.
    
- **Deep Understanding:** No blind copy-pasting. Every config line, script, or registry switch is tested, researched, and documented. It takes time, but it's always worth it.
    
- **Hardware-Constrained:** Running local LLMs (Mistral 7B) on pure CPU clusters (Dual Xeon) through strict memory management and optimized pipelines.

--- 
## Summary

- [Attached Pieces](Pièces%20jointes/README.md)
- [AI Server](AI%20Server/README.md)
- [Financial Market Intelligence Project(Inside Other Project)](Other%20Projects/Financial%20Market%20Daily&Monthly%20Intelligence/README.md)
* [KaramelIA](KaramelIa/README.md)
* [n8n Projects (Inside Other Projects)](Other%20Projects/n8n%20projects%20(automations)/README.md)
* [Other Projects](Other%20Projects/README.md) (The Unstarted)
* [Russian Self-Learning](Russian%20Self-Learning/README.md)
* [Scripting](Scripting/README.md)
* [Self-Learning](Self-Learning%20By%20Myself/README.md)