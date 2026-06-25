# AI Server Deployment and Automated AI-CyberAgent Logs Analyzer

> **Documentation of a sovereign Free-Open Source Stack (FOSS) infrastructure and an n8n workflow designed for automated system logs analysis and remediation.**

---

## Vision & Purposes

This repository tracks the deployment of a self-hosted AI infrastructure and the creation of a smart cyber agent (IRAS) in charge of filtering, transforming, and analyzing syslogs and auth.logs. The main goal is to reduce the gap between the idea and the execution using a fully on-prem FOSS stack, keeping everything independent from third-party cloud APIs.

- **Skill Acquisition:** Understanding the raw, underlying interactions between network protocols and core tools (Docker, Loki, Vector, n8n, Ollama).
- **Low ressources, High use** Strict resource optimization to run LLM inference on dual-CPU architectures due to a lack of a GPU.
- **Methodology (The Grind):** Documenting the "why" and "how" behind every single node and script to make sure we build from deep understanding, not just imitation.

---

## The Technical Core (Tech Stack)

### The Automation & AI Hub
- **The Central Nervous System:** n8n — Runs as the core orchestrator. It schedules the log gathering cycles every 15 or 30 minutes, processes the ETL logic, and handles the alerting downstream.
- **The LLM Engine:** Ollama + LiteLLM — Ollama runs as the AI "Play Store" and execution engine. LiteLLM acts as the translator between software applications and Ollama, keeping everything in a standard OpenAI API format.
- **The Interface:** OpenWebUI — Used to manage custom AI profiles, chat, or ingest documents to work on.
- **The Security Proxy:** Caddy Server — Installed to bypass Let's Encrypt limitations. It handles clean HTTPS out-of-the-box with internal SSL certificates to secure session cookies on n8n and OpenWebUI.

### Telemetry & Log Ingestion Pipeline
- **Collection & Remapping:** Vector Aggregator — Collects raw logs from its endpoints (via http_server and syslog sources) and normalizes Linux and Windows events into a single, clean JSON structure.
- **Log Storage:** Grafana Loki — Time-series log storage utilizing a TSDB index and a local filesystem layout. Configured with a built-in compactor for an automatic 14-day data retention policy (retention_period: 336h).
- **Hardware Monitoring:** Prometheus + NodeExporter + Grafana — Deployed natively using host network mode to scrape server metrics (port 9100) and display them through Grafana (Dashboard ID: 1860).

### Hardware Specifications
- **Host Server:** Dell Server running an ESXi Hypervisor
- **OS:** Ubuntu Resolute Raccoon 26.04 (CLI only)
- **Resources:** 2 Intel Xeon 2.10Ghz | 256GB RAM (Ollama container memory limit hard-capped at 150GB) | 8TB Storage running a Raid 5 Replication Storage array.

---

## Architecture Layout & Data Flow

```text
[SRV-LNX-01..N]  ──► Vector Agent ──┐
[SRV-WIN-01..N]  ──► Vector Agent ──┤
                                    ▼
                          [Vector Aggregator] (Port 9000) ──► Remap & Enrich JSON
                                    │
                                    ▼
                                 [Loki] (Port 3100) ──► TSDB Index / Fs Storage
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
   [Grafana UI]                                      [n8n Schedule Trigger]
(Dashboard Code 1860)                                   Interval: 15/30 min
                                                               │
                                                               ▼
                                                     HTTP GET Request (Loki API)
                                                       query_range & nanoseconds
                                                               │
                                                               ▼
                                                    [Code JS : Parser + Prompt]
                                                    ETL, Null-Check, Context Slice
                                                               │
                                                               ▼
                                                    [HTTP POST : Analyse Mistral]
                                                    Local Inference (stream: false)
                                                               │
                                                               ▼
                                                    [Code JS : Parser Réponse IA]
                                                    Regex split, Routing, Subject
                                                               │
                                               ┌───────────────┴───────────────┐
                                               ▼                               ▼
                                       (is_critical = true)            (is_critical = false)
                                               │                               │
                                               ▼                               ▼
                                         Instant Mail                   Cyclic Report
                                        Critical Alert                 Standard Summary
                                        
                                        
