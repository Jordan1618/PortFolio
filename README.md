# 🌐 Jordan's CV Hub & Portfolio

> **Welcome to my local knowledge base. This repository tracks my learning journey, showcases production infrastructure deployments, custom AI automations, and side-projects built with a deep-understanding approach. I wrote this whole README myself, I wanted to make it more personal and very representative of who I am**

---
## 📬 Contact

- **GitHub:** [github.com/Jordan1618](https://github.com/Jordan1618)
- **LinkedIn:** [Jordan P.](https://www.linkedin.com/in/jordan-p-77a697228/)
- **Email:** jordan.poncetpro@gmail.com

---
## 🚀 Start Here

If you only have five minutes, read these three:

- **[AI Server](Projects/AI%20Server/README.md):** a self-hosted AI stack on a dual-Xeon server with no GPU. Linux and Windows logs go through Vector and Loki, a local Mistral model analyzes them in n8n, and critical alerts reach me by mail. Everything stays on-prem, with no third-party cloud API.
- **[Instagram Messages Analytics](Projects/Instagram%20Messages%20Analytics/Instagram%20Messages%20Analytics.md):** a local pipeline that turns my own Instagram export (about 446,000 messages and 46,000 voice messages) into four reports, with Whisper transcription on my own laptop. Nothing is uploaded.
- **[My Own Tools](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/README.md):** PowerShell tools I wrote to fix real problems at work: silent .NET / VC++ updates with email alerting, and an update unblock script for locked machines.

---
## 🎯 My Vision and My Skills

> I don't do things halfway. If I start learning something, I want to understand it down to the wiring, not just enough to sound competent about it. Half-understood knowledge bothers me more than not knowing at all, that's the actual reason this vault exists.
> 
> To be honest, right now I'm building the skill to deploy AI on infrastructure that answers to no one but its owner: sovereign, legal under EU rules, and genuinely useful, not just impressive in a demo. Cybersecurity, compliance, systems, automation, none of that feels like separate boxes to me, it's the same curiosity pointed in different directions depending on the week.
> 
> IT is too big a world to fake your way through, so I let curiosity pick my direction instead of a roadmap. Skills, knowledge, and the people I build things with, that's what I'm actually optimizing for. That's also why this vault is "English-Full", I'd rather train the language while I train everything else.
> 
> My main projects are : 
> **Technological intelligence agent (Personalized News Collector and Aggregator)/Sovereign AI Deployment (Log analyzer on a Production Infra) and various personal projects (KaramelIA/Self-Learning Documentations/Tailored Financial assets tracking/Instagram Messages Analytics).**
> 
> *For context: I started in IT in August 2025, and this portfolio has been running since around April 2026, roughly when KaramelIa started too. Everything here has been built in that window.*

---
## 📂 Project Directory & Portfolio Index

### 🖥️ [AI Server (Sovereign Local AI Stack)](Projects/AI%20Server/README.md)

This one runs alongside my actual job. I use it to push toward the skills I want to reach in infrastructure and sovereign AI, hardening a stack the same way I'd want to do it professionally, layer by layer, so that by the time I'm asked to do it for real, I've already done it once for myself.

### 💼 [FreeLance Activity](Projects/FreeLance%20Activity/README.md)

This ties back to my micro-enterprise on the side. I want to be taken seriously as a professional and show real interest in where tech is heading, not just where it's been. PolyProject1 is the clearest example: building multi-agent architecture with other people, on something real, while growing a bit of extra income from skills I actually trust myself to use.

### 🛠️ [My Own Tools - Cheat Sheets](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/README.md)

This is where learning turns into something I can actually share with someone else. I build open-source tools and complete solutions, not just to have them, but because building the whole thing helps me to understand the mechanics. If I can't build a working version of something, I haven't really understood it yet.

A few examples taken from my repo :
- [.NET / VC++ auto-update script](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/Tool%20---%20Automated%20Updates%20V2%20AspCoreNetRuntimeVC.md)
	Checks and silently updates .NET runtimes and VC++ redistributables across machines, with email alerting on failure
- [Windows Updates unblock & hardening script](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/Tool%20---%20Unblock%20Windows%20Updates%20V2%20%28Strengthen%29.md)
	Self-elevating cleanup and policy reset for machines with blocked update settings
- [Windows 11 25H2 hardware-check bypass script](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/Tool%20---%20Inject%20Strengthen%20Keys.md)
	Registry-level bypass (TPM/CPU/RAM/Secure Boot checks) plus auto-detection of a mounted install ISO, for upgrading otherwise-blocked hardware
	
	Nb : Done on old hardware at my place. Security stays my main purpose

### 📊 [Projects](Projects/README.md)

A mix of practical and personal: numbers that have to be right (Financial Market Intelligence), automations I'd rather build once than repeat by hand, a data project on my own messages (Instagram Messages Analytics), and a to-do list of ideas I haven't started yet, honestly the folder that scares me the most. **KaramelIa** is in here too: it started as a small experiment with AI music tools, and it's slowly become a real project, with its own visual identity and production process.

Everything planned or paused (including the empty folders) is listed in one place: [Roadmap - Everything I Plan To Do](Projects/Projects%20To%20Do/Roadmap%20-%20Everything%20I%20Plan%20To%20Do.md).

**KaramelIA YouTube Link :** https://www.youtube.com/@KaramelIa_Music

### 🧠 [Self-Learning By Myself (Core Knowledge Vault)](Self-Learning%20By%20Myself/README.md)

The closest thing I have to a personal curriculum, organized around what I actually want to get good at:

- **Networks and protocols**, because I need to understand how my data travels and changes from point A to point Z
- **Systems and infrastructure**, because I want to know how things are working and be "The person who knows"
- **Security and anonymity**, because understanding how to break something is the only way to defend it
- **Scripting and automation**, because I like that. Automating everything is a key skill and I want to learn how to make it mine + it's a proof of work
- **Digital identity**, because it is necessary in today's world
- **Compliance (RGPD, NIS2)**, because that's exactly what I want to become : The person who knows how the engine turns and legally and efficiently. *This folder is still empty on purpose, it is planned in the [Roadmap](Projects/Projects%20To%20Do/Roadmap%20-%20Everything%20I%20Plan%20To%20Do.md)*
- *And underneath all of it, a running library of whatever I've already learned or still want to, wherever curiosity decides to point next*

## 🛠️ Technical Skills & Tools Stack

- **Infrastructure:** VMware ESXi, Linux Ubuntu/Debian CLI, Windows Server
- **Containers & Admin:** Docker, Docker Compose, systemd, Windows Registry, GPO, Active Directory, RDP
- **Logs & Telemetry:** Vector (VRL), Grafana Loki
- **Monitoring:** Prometheus, NodeExporter, Grafana Dashboards
- **Network & Security:** UFW, Caddy Server (Reverse Proxy & TLS), Network Segregation, Firewalls, SIEM, EDR/NDR
- **Automation:** n8n, JavaScript, ETL Pipelines
- **Local AI:** Ollama (Mistral 7B, Mixtral), LiteLLM Proxy
- **Data & Local Speech-to-Text:** Python, faster-whisper (Whisper), NumPy
- **AI-Assisted Dev:** Claude Code (desktop, CLI and VS Code)

### 📎 [Pièces jointes](Pi%C3%A8ces%20jointes/README.md)

The receipts. Supporting files, screenshots, and references that back up everything else in this vault.

### 🧩 [Skills & MasterPrompts](Tools%20&%20Skills/Skills%20&%20MasterPrompts/README.md)

A hand-shaped, open-source library, nothing fancier than that. Reusable frameworks and prompt systems I put together so anyone, including future me, can actually pick them up and use them, not just admire them.

A few examples of my best Skills for AI :
- [Portfolio Tech Notes](Tools%20&%20Skills/Skills%20&%20MasterPrompts/Skills/Portfolio%20Tech%20Notes.md): writes my vault documentation in three fixed formats (Course, Cheat Sheet, README) and corrects my English
- [Self-Help Book Learning](Tools%20&%20Skills/Skills%20&%20MasterPrompts/Skills/Self-Help%20Book%20Learning.md): learn a book by practicing real situations, not by reading a summary
- [English Notes Skill](Tools%20&%20Skills/Skills%20&%20MasterPrompts/Skills/English%20Notes%20Skill.md): corrects my English and tracks my mistakes in [Mistakes Learned](Side%20Learning/English%20Self-Learning/Mistakes%20Learned.md)
- Instagram Messages Analytics skills: two private skills I wrote for that project, not published

---
## 🎯 Strategic Principles

- **Understanding Data, Not Just Storing It:** Knowing how data actually flows, where it goes, and what happens to it at each step, that's the base I'm building everything else on.
- **EU-Compliant AI Infrastructure:** Learning to deploy AI infrastructure that actually holds up against RGPD and NIS2, not just technically capable but properly compliant.
- **Deep Understanding:** No blind copy-pasting. Every config line, script, or registry switch is understood, and documented. It takes time, but it's always worth it.
- **Where I'm Headed:** All of this is building toward a VIE or a well-paid real job in my path if I can get there, applying these skills for real.

---
## 🌱 Personal / Side Learning

### 🇷🇺 [Russian Self-Learning](Side%20Learning/Russian%20Self-Learning/README.md)

Learning Russian was never a career move, it was something that interested me. I wanted proof I could hold a structured discipline on something nobody is paying or grading me for. It is paused for now, because I focus on other projects, but I did learn some: the A1 foundations are done and the full A1 to C2 roadmap is mapped for when I have time.

--- 
## Summary

- [AI Server](Projects/AI%20Server/README.md)
- [English Self-Learning](Side%20Learning/English%20Self-Learning/README.md)
- [FreeLance Activity](Projects/FreeLance%20Activity/README.md)
- [My Tools And Cheat Sheets](Tools%20&%20Skills/My%20Own%20Tools%20-%20Cheat%20Sheets/README.md)
- [Pièces jointes](Pi%C3%A8ces%20jointes/README.md)
- [Projects](Projects/README.md)
- [Roadmap - Everything I Plan To Do](Projects/Projects%20To%20Do/Roadmap%20-%20Everything%20I%20Plan%20To%20Do.md)
- [Russian Self-Learning](Side%20Learning/Russian%20Self-Learning/README.md)
- [Self-Learning By Myself](Self-Learning%20By%20Myself/README.md)
- [Skills & MasterPrompts](Tools%20&%20Skills/Skills%20&%20MasterPrompts/README.md)

Nb : Sometimes you will see French. It's normal, I'm French and some examples can contain a little part of that. Be sure, I speak fluent English and I'm learning more and more with each new document
