# 🌐 Jordan's CV Hub & Portfolio

> **Welcome to my local knowledge base. This repository tracks my learning journey, showcases production infrastructure deployments, custom AI automations, and side-projects built with a deep-understanding approach. I did all this README by myself, I wanted to make it more personal and very representative of who I am**

---
## 🎯 My Vision and My Skills

> I don't do things halfway. If I start learning something, I want to understand it down to the wiring, not just enough to sound competent about it. Half-understood knowledge bothers me more than not knowing at all, that's the actual reason this vault exists.
> 
> To be honest, Right now I'm building the skill to deploy AI on infrastructure that answers to no one but its owner: sovereign, legal under EU rules, and genuinely useful, not just impressive in a demo. Cybersecurity, compliance, systems, automation, none of that feels like separate boxes to me, it's the same curiosity pointed in different directions depending on the week.
> 
> IT is too big a world to fake your way through, so I let curiosity pick my direction instead of a roadmap. Skills, knowledge, and the people I build things with, that's what I'm actually optimizing for. That's also why this vault is "English-Full", I'd rather train the language while I train everything else.
> 
> My main projects are : 
> **Intelligence technologic agent(Personalized News Collector and Aggregator)/Sovereign AI Deployment (Log analyzer on a Production Infra) and various personal projects (KaramelIA/Self-Learning Documentations/Financial assets on-mesure tracking).**

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
    
- **Others:** Claude Caude (desktop and vscode)
    

## 🎯 Strategic Principles

- **Absolute Data Sovereignty:** Zero logs, tokens, or network data ever leave the local network. Everything is 100% local and airtight.
    
- **Deep Understanding:** No blind copy-pasting. Every config line, script, or registry switch is tested, researched, and documented. It takes time, but it's always worth it.
    
- **Hardware-Constrained:** Running local LLMs (Mistral 7B) on pure CPU clusters (Dual Xeon) through strict memory management and optimized pipelines.

--- 
## Summary

- [AI Server](AI%20Server/README.md)
- [English Self-Learning](English%20Self-Learning/README.md)
- [FreeLance Activity](FreeLance%20Activity/README.md)
- [My Tools And Cheat Sheets](My%20Own%20Tools%20-%20Cheat%20Sheets/README.md)

Nb : Sometime you will see French. It's normal, I'm French and some example can contain a little part of that. Be sure, I speak a fluent English and I'm learning more and more at each new documentation