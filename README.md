# 🌐 Jordan's Engineering Hub & Sovereign Portfolio

> **Welcome to my local knowledge base. This repository tracks my continuous learning journey, shows production-ready infrastructure deployments, custom AI automations, and side-projects built with a deep-understanding approach.**

---
## 🎯 Executive Profile & Core Vision

> **Aspiring Technology Watch Officer (Veilleur Technologique) specializing in Sovereign AI Deployment and European Regulatory Compliance.**
> 
> My long-term professional objective is to help organizations architect, audit, and deploy IT infrastructures and localized AI systems that are 100% compliant with European Union regulations (RGPD, EU AI Act). I am dedicated to proving that high-performance automation and state-of-the-art Local Intelligence can perfectly coexist with data privacy, intellectual property rights, and ethical digital sovereignty.

---
## 📂 Project Directory & Portfolio Index

### 🛡️ Enterprise-Grade Cybersecurity & Local AI Stack
* **[[1) AI Server Deployment and an A-to-Z explanations|1) AI Server Deployment & Hardening]]** : Technical architecture of a sovereign inference server running on corporate-grade hardware (Dell Server under ESXi, 2x Intel Xeon, 256GB RAM). Comprehensive documentation covering Docker deployment, strict network isolation (`ai-internal`), and resource hardening with a hard-capped memory allocation of 150GB to mitigate Log Flooding/Denial of Service (DoS) risks.
* **[[2) AI Server Automated AI-CyberAgent logs analyzer|2) AI Server Automated CyberAgent Logs Ingestion]]** : End-to-end design of an automated log ingestion pipeline. Configuration of lightweight Vector agents for semantic normalization of Windows Security Logs (EventID 4624) and Linux syslogs sent to a centralized Grafana Loki log database.
* **[[3) AI n8n Workflow More Detailed Paper|3) AI n8n Crisis Orchestration & Data Parsing]]** : Software engineering applied to crisis orchestration. Deep-dive into custom JavaScript parsing nodes executed inside n8n (handling null-checks, text-slicing for context-window constraints, and JSON payload restructuring) to format raw data before submitting it to the localized LLM.
* **[[4) Deploying Agent on a Windows Server|4) Remote Telemetry Deployment]]** : Multi-platform telemetry deployment automation. Design and configuration of remote Vector sensors on production Windows servers for real-time security tracking.
* **[[0) Conf Files For Global Looking|5) Infrastructure-as-Code Configuration Master Repo]]** : Centralized, structural repository hosting production-ready Infrastructure-as-Code setups: master `docker-compose.yml` configurations, VRL transformation rules (`aggregator.toml`), and LiteLLM security proxy specifications (`litellm_config.yaml`).

### 📊 Financial Data & Market Analytics
* **[[Financial Market Daily&Monthly Intelligence/README|Financial Market Daily & Monthly Intelligence]]** : Engineering an automated quantitative and qualitative data pipeline targeting global financial assets. This project focuses on the collection, structured parsing, and execution of automated reporting scripts to generate periodic macro-market summaries and asset risk analytics through autonomous localized routines.

### 🧠 Advanced Self-Learning & Knowledge Documentation
* **[[Self-Learning By Myself/README|Core Knowledge Vault & Self-Learning Engine]]** : The absolute core and largest segment of my portfolio. This master repository aggregates my deep-dive documentation, research notes, and structured technical summaries on emerging systems, infrastructure paradigms, and computer science fundamentals, showcasing a non-stop, disciplined approach to technological mastery.
* **[[Scripting/README|Systems Scripting & Lower-Level Automation]]** : Production automation environment gathering lightweight system administration tools, local batch utilities, cross-platform data parsing workflows, and cron-based maintenance routines.
* **[[Unblock Windows Updates .bat|Windows OS Remediation Utility]]** : Advanced Windows administrative shell script. Specifically designed to bypass local UAC restrictions for standard unprivileged users, safely manipulate specific key values within the `HKLM` registry branch, and enforce a hardened, automated maintenance scheduler targeting mandatory updates at 8:00 PM.

### 🎨 Creative AI Engineering & Content Pipeline
* **[[KaramelIa/README|KaramelIa Automated Content System]]** : Technical and strategic direction of an autonomous, multi-platform multimedia content generation system driven by customized AI workflows.
* **[[Project Evolution and Documentation|Production Logs & Video Processing Pipeline]]** : Technical dev logs tracking the optimization of complex video processing pipelines, transition to professional editing frameworks (DaVinci Resolve), and integration of automated n8n publishing sequences mapping dynamic SEO generation, descriptions, and scheduling across YouTube, TikTok, and Instagram.

### 🇷🇺 Continuous Disciplined Linguistics
* **[[Russian Self-Learning/README|Structured Language Acquisition Framework]]** : Highly disciplined workspace dedicated to mastering the Russian language. Tracks personal progress frameworks, lexical databases, complex grammatical deep-dives, and objective assessment matrices designed to clear milestones from beginner to advanced intermediate proficiency (A1 to B2 targets).

---

## 🛠️ Global Technology & Hardening Matrix

| Infrastructure Layer | Technology Implemented | Core Operational Role | Hardening & Security Controls |
| :--- | :--- | :--- | :--- |
| **Operating System** | Ubuntu 26.04 CLI LTS | Base Server Environment | UFW firewall strict subnet restriction (`192.168.1.0/24`), forced SSH keys, automated security patching. |
| **Container Engine** | Docker CE / Compose | Micro-service Isolation | Non-root runtime contexts, memory hard-caps (`limits.memory: 150g`), isolated internal communication networks. |
| **API Management** | LiteLLM Proxy Gateway | Unified Endpoint Mapping | Authentication tokens enforced, zero outside telemetry mode, strict timeout configurations. |
| **Local Inference** | Ollama Engine | Core Intelligence Execution | Isolated in an internal-only bridge network (`ai-internal`), zero external API calls. |
| **Log Management** | Vector Aggregator | ETL Log Parsing & Shipping | Port access restrictions, input size limitation to block log flooding attempts. |
| **Log Database** | Grafana Loki | High-Performance Storage | Enforced file-system permissions, automatic data expiration and rolling retention policy. |
| **Traffic Routing** | Caddy Server | HTTPS Reverse Proxy | Fully automated TLS/SSL lifecycle handling, secure session headers, local reverse-proxy containment. |

---

## 🎯 Strategic Principles & Engineering Mindset

* **Absolute Data Sovereignty:** Zero logs, tokens, credentials, or network topologies ever exit the local infrastructure. The entire stack is built to be strictly airtight, adhering to the stringent compliance expectations of defense and government-grade deployments.
* **The "Deep Understanding" Core:** Complete rejection of unverified copy-pasting. Every configuration parameter, custom JavaScript ETL block, registry switch, or bash operation is systematically researched, tested, and documented.
* **Hardware-Constrained Engineering:** Proven capacity to architect resilient, high-availability monitoring layers, adapting complex localized LLM execution (Mistral 7B) to pure CPU computation clusters (Dual Intel Xeon) via rigorous memory management and bounded data ingestion pipelines.