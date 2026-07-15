
# 📊 Financial Market Daily & Monthly Intelligence

## 🚧 Status: Work In Progress (Active Production Mapping)

> **This module implements a hardened, cloud-isolated financial intelligence pipeline designed for long-term macro and micro asset tracking. Built with strict structural boundaries, it isolates private asset intelligence from institutional lab hardware.**

---

## 🎯 Project Scope & Core Cycles

The system acts as an automated autonomous analyst, engineered for long-term value investing while avoiding daily short-term market noise. It operates across three tightly scheduled routines:

1. **Daily Operational Pipeline (Mon-Fri @ 07:30 AM):** Low-overhead cron sequence fetching closing values and core economic shifts. It compiles data into a highly synthesized, responsive layout optimized for fast reading during morning transit commutes.
2. **Weekly Strategic Review (Saturdays @ 08:00 AM):** Executed once markets are closed. Gathers the week's logs, processes data aggregates, and leverages localized European AI models to output a deep structural analysis on long-term trends and potential asset under-valuations.
3. **Monthly Governance Portfolio Audit (1st of Each Month):** Triggers a secure configuration loop to recalibrate definitive portfolio weights, cost-basis changes, and monitors upcoming corporate Annual General Meetings (AGMs) for physical shareholder attendance and real-world networking.

---

## ⚙️ Hybrid Technical Architecture

To guarantee absolute reliability, the workload is bifurcated based on engineering best practices:

* **The Deterministic Core (No-LLM Math):** Core asset ledgers, performance differentials, and percentage gains are strictly calculated via native JavaScript scripts. This guarantees mathematical integrity and completely eliminates AI hallucinations on financial metrics.
* **The Semantic Layer (EU-Friendly AI):** European-hosted open models (Mistral AI APIs) are used exclusively for text summarization, macro-economic news correlation, and multi-source context synthesis during the weekend cycle.
* **Infrastructure Isolation:** Hosted on an external, lightweight containerized cloud stack. This enforces strict administrative and ethical boundaries, ensuring zero overlap with public infrastructure systems.

---

## 🛠️ Data Flow & Storage Blueprint

```

[Time Trigger] ──► [n8n Orchestrator (Cloud Node)] ──► [HTTP Financial APIs]

│

▼

[SMTP Mail Notification] ◄── [JS Parsing & Processing] ◄── [Local JSON/SQLite Storage]

│

└──► (If Saturday) ──► [Secure European Mistral API] ──► [Obsidian Note Output]

```

---

## 📅 Roadmap & Milestones

- [x] Functional requirements definition and market schedule blueprinting.
- [x] Ethical framework validation & cloud migration architecture choice.
- [/] Data schema definition for asset inputs and AGM calendar tracking.
- [ ] Active n8n workflow deployment (JSON processing nodes & SMTP integration).
- [ ] Prompt validation for Saturday's macro-economic structural reporting.