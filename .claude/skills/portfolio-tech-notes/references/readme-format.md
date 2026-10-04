# README format

A README is the index/overview page for a folder in Jordan's vault. It never contains the actual technical content (no commands, no step-by-step): it explains what the folder is for and links to what's inside. There are two tiers. Picking the right one matters more than anything else in this file.

## Which tier?

- **Tier A, Index README**: the folder just organizes a set of existing Course / Cheat Sheet / Tool notes. No architecture of its own, nothing was "built". Examples: `My Own Tools - Cheat Sheets`, `Self-Learning By Myself`, `4 - Scripting & Automation`.
- **Tier B, Project README**: the folder documents an actual project/build with its own tech stack and, usually, several linked "parts". Examples: `AI Server`, and likely future entries under `Projects` (KaramelIa, n8n projects) or `PolyProject1`.

If genuinely unsure which one fits, ask Jordan rather than guessing. Don't inflate a simple topic folder into a fake "project" README, and don't flatten a real build into a plain link list.

## Tier A: Index README

```
# Title of the folder

**One-sentence bold tagline: what this folder is for.**

---
## Main Topics
(1-2 sentences: what's centralized here)

---
## Goal of this Documentation
*   **Track Progress:** Keep a clear record of ... (adapt to the folder)
*   **Quick Reference:** Create a library of reusable ...
*   **Deep Understanding:** Force myself to ...

---
## [Summary / Topics / Toolbox, pick whichever label matches the folder]
- [Display name](file%20name%20url%20encoded.md)
- [Display name](file%20name%20url%20encoded.md)

---
```

- `---` separates every section: right after the title/tagline, and around each `##` block.
- The three "Goal of this Documentation" bullets are boilerplate Jordan reuses almost word-for-word across index READMEs, reuse them and only adapt the folder-specific detail inside each one. Skip this whole section for a small subfolder README that's just a flat list of course links (see the second worked example below).
- If the folder mixes types of content (e.g. "Tools" vs "Cheat Sheets"), split into separate `##` sections instead of one flat list.
- Link syntax: standard markdown link, spaces in the real filename url-encoded as `%20`, no invented prefix. The link text must match the actual filename exactly.

### Worked example: full index README (real note, trimmed)

```markdown
# Automation & System Tools

**Documentation of everything I craft and deploy to eliminate repetitive tasks, optimize environments, and accelerate my technical capabilities.**

---
## Main Topics
This directory centralizes my custom automation scripts, registry configurations, and dives into shell scripting.

---
## Goal of this Documentation
*   **Track Progress:** Keep a clear record of how my tools evolve from basic commands to complex architectures.
*   **Quick Reference:** Create a library of reusable code blocks, functions, and logic gates to deploy tools faster.
*   **Deep Understanding:** Force myself to break down every single line of code and master the underlying software or protocols.

---
## My Own Tools
- [Tool --- Inject Keys](Tool%20---%20Inject%20Keys.md)
- [Tool --- Unblock Windows Updates V1](Tool%20---%20Unblock%20Windows%20Updates%20V1.md)

## My Cheat Sheets
- [Cheat Sheet Windows Tools Essentials (Win+R)](Cheat%20Sheet%20Windows%20Tools%20Essentials%20(Win+R).md)
---
```

### Worked example: light subfolder README (real note, trimmed)

A subfolder that just links its own course notes plus related cheat sheets elsewhere can skip Main Topics / Goal of this Documentation entirely:

```markdown
# Scripting | System Tools Linked

**Documentation of everything I craft and deploy to eliminate repetitive tasks, optimize environments, and accelerate my technical capabilities.**

---
## Summary

- [Auto-Update .asp-.net-.core-VC+](Auto-Update%20.asp-.net-.core-VC+.md)
* [PowerShell Fundamental Knowledge](PowerShell%20Fundamental%20Knowledge.md)

## Complementary Documentation :

- [Cheat Sheet Windows Tools Essentials (Win+R)](Cheat%20Sheet%20Windows%20Tools%20Essentials%20(Win+R).md)
```

## Tier B: Project README

```
# Project Title

> **One or two sentence bold summary of what this is.**

---
## Vision & Purposes
(short paragraph: the problem this solves, why it was built this way)
- **[Angle 1]:** ...
- **[Angle 2]:** ...
- **[Angle 3, methodology]:** ...

---
## The Technical Core (Tech Stack)
### [Sub-system name]
- **[Component]:** what it does, in plain terms.

### Hardware Specifications (if relevant)
- ...

---
## Architecture Layout & Data Flow
```
(ASCII diagram of the flow: arrows only, no prose inside the block)
```

---
## Summary
* [Part name](file.md)
```

- Only use Tier B when the folder genuinely has multi-part documentation and its own architecture. Don't invent Tech Stack/Architecture sections for a simple topic.
- The ASCII diagram is optional and only goes in if Jordan gives or confirms the actual data flow. Never invent one.
- Tech Stack sub-headers must mirror the real components being documented, not a generic list.

## Common mistakes to avoid

- Do NOT write actual technical instructions/commands inside a README. That belongs in the Course/Cheat Sheet/Tool doc it links to.
- Do NOT invent links to files that don't exist. Only link files Jordan confirms are in that folder.
- Do NOT mix tiers on the same README. A simple index folder doesn't need Tech Stack/Architecture, and a real project shouldn't be flattened into just a link list.
- Filename is always `README.md`, no exceptions.
