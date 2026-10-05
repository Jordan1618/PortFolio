# Roadmap - Everything I Plan To Do

**One place for everything that is planned, paused or not started yet. If a folder or a note looks empty somewhere else in this vault, the reason is probably here.**

---
## How to read this page

- **Planned:** I want to do it, nothing is written yet.
- **In progress:** started, not finished.
- **Paused:** started, then stopped on purpose to focus on something else.
- **Done but not documented:** I did it at work or on my own, the note is still missing.

---
## 1) Projects

| Project | Status | Where | Next step |
|---|---|---|---|
| AI Server (logs analyzer) | In progress | [AI Server](../AI%20Server/README.md) | Windows agent deployment (4th part), Windows event logs into Loki |
| KaramelIa | In progress | [KaramelIa](../KaramelIa/README.md) | Automate the publishing flow with n8n, music distribution, TikTok and Instagram accounts |
| Financial Market Intelligence | Planned | No folder yet | Data schema, n8n workflow deployment, prompt validation for the Saturday report |
| Cyber Intelligence Newsletter (n8n) | Planned | No folder yet | Weekly global cyber edition on top of the Monday/Wednesday/Friday ones |
| Instagram Messages Analytics | Done, rerunnable | [Instagram Messages Analytics](../Instagram%20Messages%20Analytics/Instagram%20Messages%20Analytics.md) | Finish the last voice messages, maybe a small dashboard |
| Public-interest meeting platform | Idea | [See section 6](#6-idea-public-interest-meeting-platform) | Test the idea on a small scale first |


### Not started (only a name for now, no folder on purpose)
- Auto PortFolio adder for my YouTube last watched videos
- Automated Market Studies by AI FOSS
- HTB CTF
- Idea Aggregator For Companies
- PromptShelves
- Public Interest and Ethical Dating App (see [section 6](#6-idea-public-interest-meeting-platform))
- WorldCyberWatch
- Financial Market Daily & Monthly Intelligence
- n8n Automation Lab (cross-project workflows)
- n8n: Cyber Intelligence Newsletter (AI-built)
- n8n: Cyber Threat Intelligence
- n8n: Next Idea to Come
- n8n: YoutubeWrapped

---
## 2) Self-Learning topics not written yet

### Compliance & Governance (folder is empty on purpose)
- NIS2
- RGPD
- What a "conform" AI is under the European digital acts
- How to run a perfectly legal AI server in the EU

### Done but never documented
- AD / GPO
- Firewall
- RegEdit / Windows Update
- SIEM / EDR-NDR / Antivirus
- Phishing campaigns
- TeamViewer GPO updates

### Protocols and concepts to master
- Kubernetes
- SSL/TLS certificate expiration
- Shadow IT
- NAC
- UEFI / BIOS
- Linux kernel and Windows kernel
- IIS
- How logs work
- YAML / TOML
- TypeScript
- Go

---
## 3) Technical ideas

### Security and infrastructure
- Nmap tool to check if my infrastructure is compromised or backdoored, and to check my attack surface
- User access review automation with scripting, displayed in Grafana
- UFW firewall rule auditing and logging
- SSH banner customization and legal warning
- SSH brute-force automated IP banning workflow
- Docker container escape auditing and hardening
- Automated honeypot deployment with an n8n trigger
- Offline CVE vulnerability matching pipeline
- Ransomware behavior detection with file canaries
- PowerShell Constrained Language Mode deployment
- Active Directory Kerberoasting remediation
- Memory forensic analysis on hardened AI hosts
- Hardware temperature and CPU alerts in Grafana, with automatic shutdown if too dangerous

### AI
- Hardening a self-hosted AI server
- RAG pipelines security risks
- Local RAG document expiry and data lifecycle automation
- Local LLM model weight integrity verification
- API rate-limiting architecture for LiteLLM gateways
- Vector VRL rules for corporate data masking
- Prompt injection tests, locally
- Custom AI system prompt for an IT helpdesk role
- LLM benchmarks for automation, and a comparison of AIs for each use case
- Fine-tuning

### n8n workflows
- Email notification on a webhook trigger
- Automated Git backup for the Obsidian vault
- Server disk space monitor via webhook
- Automated social media trend alerts
- Multi-source incident correlation engine
- Automated vulnerability scanner and patch pipeline (nmap, open ports, CVE matching, alert with the patch method)
- Automated Threat Intelligence workflow (CVE databases, data leaks, new products)

### Small scripts and cheat sheets
- Disk space alert script for Ubuntu Server
- Automated ping test for switches, servers and firewall
- Windows service status checker with PowerShell
- Linux permissions and commands cheat sheet
- Windows Event Viewer
- Analysis of a simple phishing email

---
## 4) Paused

- **Russian:** paused to focus on other projects. A1 foundations are done, the rest of the roadmap is mapped but not started. See [Russian Self-Learning](../../Side%20Learning/Russian%20Self-Learning/README.md).

---
## 5) Other

- Harvard open classroom on Data Science
- Family office (business and patrimony)
- Same method as Instagram Messages Analytics for WhatsApp / Telegram exports, support chats and call recordings

---
## 6) Idea: public-interest meeting platform

**The idea:** a non-commercial platform that helps people meet in real life, with a public-interest goal: reduce loneliness and the mismatches created by apps that live on advertising and paid features.

**Principles I would not give up**
- No advertising and no paid boosts inside the app.
- A self-sufficient model, based on stable and predictable resources instead of ad revenue.
- Verified identities and verified pictures, to keep the platform safe.
- Private conversations, and a simple way to report inappropriate content.
- Inactive accounts are closed after a long absence, to keep the community alive.
- Balanced access between groups, managed with a waiting list.

**Open questions**
- How to finance it: public funding alone will probably not be enough, an association status may be needed.
- How to keep the balanced access rule fair, because it can be seen as indirect discrimination.
- How to handle a large amount of personal data in a way that respects RGPD.
- What to offer users who get few matches, in a way that is useful and respectful.
- How to avoid any misuse of such a platform by public authorities.

**Next step:** test the concept on a small scale first (one city, or one large company where people do not know each other), to find the limits and a first compromise.
