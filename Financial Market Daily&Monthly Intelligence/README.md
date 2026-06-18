# 📊 Financial Market Daily & Monthly Intelligence

## 🚧 Status: Work In Progress (Active Development)

> **This module is currently under active construction and pipeline testing. The project aims to deliver a hardened, local, and completely sovereign financial intelligence tool designed for long-term macro and micro asset tracking.**

---

## 🎯 Project Scope & Core Targets

The objective of this sub-repository is to build an automated asset management and market tracking system operating across three distinct operational cycles:

1. **Daily Pipeline (07:30 AM Execution):** Low-overhead HTTP API pulling of market configurations, generating a highly synthesized performance email tailored for fast morning transit-reads.
2. **Weekly Review (Saturday Operations):** Cross-correlation of daily technical pricing fluctuations and global economic macro trends, analyzing underlying long-term structural market reasons rather than short-term market noise.
3. **Monthly Audit (Asset Weighting):** Manual ledger sync to recalculate the definitive weighting and cost-basis of registered shares, alongside tracking upcoming corporate Annual General Meetings (AGMs).

---

## 🛠️ Planned Core Architecture

* **Data Collection Engine:** Native `n8n` HTTP Request triggers targeting public financial endpoints (Yahoo Finance, CoinGecko).
* **Storage Options (Under Evaluation):** Testing a localized raw file layout (`JSON`/`CSV` parsing directly managed via JavaScript) versus an embedded relational schema (`SQLite` containerized micro-service).
* **Delivery Infrastructure:** Secure SMTP routing configuration to deliver crisp, responsive markdown-to-HTML templates straight to the local operator mailbox.

---

## 📅 Roadmap Milestones

- [x] Operational logic conceptualization and API endpoint mapping.
- [/] Database schema layout definition (Asset tracking and AGM calendars).
- [ ] Active n8n scheduling workflow creation and email template design.
- [ ] Production pipeline implementation and hardware metrics check on the Dell cluster.
