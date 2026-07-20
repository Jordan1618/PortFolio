## Why This Skill

Before this skill, every vault note needed the same rules re-explained from scratch each session Claude drifted into assistant-voice, added commands that were never actually run, or guessed the wrong vault folder. Once the skill was in place, Course / Cheat Sheet / README / English-correction output matches Jordan's vault style on the first try, no back-and-forth.

## Origin

### Initial Prompt

Nb : this is the earliest version I have direct visibility into the state of `SKILL.md` at the start of today's session. It's already the product of earlier corrections that aren't individually logged anywhere I can see, not the literal very first draft ever written. If you have that one, worth pasting it above this section.

```
---
name: portfolio-tech-notes
description: Writes IT/sysadmin/scripting documentation for Jordan's Obsidian portfolio vault, in one of two fixed formats — "Course" (a step-by-step narrative note about something he just learned or debugged) or "Cheat Sheet" (a short bullet-point reference list). Use this skill whenever Jordan asks to write up, document, or turn a conversation into a course, a cheat sheet, portfolio notes, or vault notes about an IT topic — even if he just says "fais-moi un doc là-dessus", "note ça", "cheat sheet sur X", or similar, without naming the format explicitly. Also use to shorten, simplify, or reformat existing course/cheat sheet drafts to match this style, and to advise where a new note should live in his vault.
---

# Portfolio Tech Notes

Generates Jordan's IT portfolio documentation in English, matching the exact tone and structure of his existing vault. He has corrected this format multiple times — treat the rules below as firm, not stylistic suggestions.

## Hard constraints (apply to both formats)
- Always English, regardless of the language Jordan writes the request in.
- Simple, student-level vocabulary...
- Short...
- Only real, necessary commands...
- Written as Jordan's own notes (first person for Courses; neutral reference tone for Cheat Sheets)...
- Output as a single .md file, created with the file tools...

## Step 1 — figure out the format
[Course vs Cheat Sheet signals, pointer to references/course-format.md and references/cheatsheet-format.md]

## Step 2 — gather the source material
[don't invent commands/steps; keep only the final correct path; ask if key details are missing]

## Step 3 — write it
[follow reference file structure exactly; self-check against constraints]

## Step 4 — advise on placement in the vault
[full vault folder map: AI Server, English Self-Learning, FreeLance Activity, My Own Tools - Cheat Sheets, Pièces jointes, Projects, Russian Self-Learning, Self-Learning By Myself (0-6), Skills & MasterPrompts — with routing rules, file naming convention, and instruction to state the suggested folder/filename alongside the file]
```

### Human Refinements

- Asked to also take into account 4 real README files pulled from the vault → added a third format, **README**, with two tiers in `references/readme-format.md` — Tier A (simple index, e.g. `My Own Tools`, `Self-Learning By Myself`) vs Tier B (project with its own tech stack/architecture, e.g. `AI Server`), so a plain topic folder doesn't get inflated into a fake "project" README and vice versa.
- Asked to merge in the separate `english-correction` skill so every rule about English in the vault — writing new docs and correcting drafted text — lives in one place → folded its content into `references/english-correction.md` and deleted the standalone skill, to avoid two skills matching the same conversation.
- Split the skill into **Part A** (writing new docs: Course/Cheat Sheet/README) and **Part B** (correcting existing text), with an explicit flag that Part B must NOT simplify Jordan's vocabulary — the opposite rule from Part A after realizing the two modes could otherwise get conflated once merged into one file.
- Extended the vault placement step to also state where each format's README tier lives (always inside the folder it indexes, filename `README.md`), instead of only covering Course/Cheat Sheet destinations.

## The Skill

Now covers 4 modes from one entry point (Step 0 routing): Course, Cheat Sheet, README (2 tiers), and English correction — each backed by its own reference file (`course-format.md`, `cheatsheet-format.md`, `readme-format.md`, `english-correction.md`). Hard constraints (English only, simple vocabulary, no invented commands/links, vault placement rules) apply to the 3 writing modes; correction mode runs on the opposite principle keep my own wording, fix only actual errors.

---
---
---
# Course format

A Course is Jordan's own debugging/learning notebook entry: what he tried, what actually happened, and what he now knows. It is not a tutorial written for someone else. It's written for Jordan-in-six-months.

## Structure

```
# **1) What I wanted to do :**
(1-3 sentences: the goal or the problem, in plain terms)

# **2) [Step or "What was really happening"] :**
(the investigation or the first attempt, with real commands)

# **3) [Next step] :**
...continue numbering through however many steps it actually took...

# **N) What I learned :**
(short, concrete takeaways, not a summary of the whole note, just the 2-4 things worth remembering)
```

**No "What to do next" / "What's next" section.** Even if the conversation with Jordan talked about future plans, ideas, or follow-up work, leave that out of the note itself. It stays between him and Claude in the chat, not in the portfolio file. The note ends on what was learned, full stop.

- Section headers are always `# **N) Title :**`, bold, numbered, colon at the end.
- Code blocks use the language tag when relevant (`powershell`, `bash`).
- A command is followed by a short explanation only if the command itself isn't obvious. Not every command needs a breakdown.
- `Nb :` on its own line introduces a side note that adds context without breaking the flow of the step. Use it sparingly: one or two per note, not after every command.
- No intro paragraph before section 1, no closing paragraph after the last section.
- First person throughout ("I wanted to...", "I checked...", "I learned...").

## Worked example (real note from Jordan's vault, lightly trimmed)

markdown

````markdown
# **1) What I wanted to do :**

I wanted to turn on Sysmon to log more details about what happens on my PC (which programs start, etc.), just to test what it changes. I found "Sysmon" as a checkbox in "Turn Windows features on or off" and ticked it. But it didn't work right away like a normal installed program.

Problems I got : Get-Service sysmon64 said "no service found with that name" Get-WinEvent on the Sysmon log said "no matching log found"

# **2) What was really happening :**

Ticking the checkbox only turns on the option. It does NOT start the program by itself, you still need one extra command. Also, I learned Windows 11 now comes with its own built-in copy of Sysmon (in `C:\Windows\System32\sysmon.exe`).

I installed the built-in one properly (PowerShell, as Administrator) :

powershell

```powershell
sysmon.exe -accepteula -i
```

Nb : `-accepteula` just skips the license popup. `-i` means "install".

# **3) Checking if it's worked :**

powershell

```powershell
Get-Service sysmon
```

Result : Status "Running". It works.

# **4) What I understood from reading the logs :**

- By default, Sysmon only logs two things : when a program starts (ID 1) and when it closes (ID 5)
- Raw logs are meant to be filtered first, not read one by one in a console
````

## Common mistakes to avoid (things Jordan has corrected before)

- Do NOT explain every command word-by-word if the command is simple (`ls`, `cd`, `cat`). Only break down commands with flags or syntax that isn't self-evident.
- Do NOT include commands that weren't actually used to solve the problem, even if they're "related" or "good practice".
- Do NOT add a generic intro like "In this note, I will show you how to...". Start directly at section 1.
- Do NOT use "you". This is Jordan's own log of what he did, not instructions to a reader.
- Keep it short: most of his real course notes are 300-500 words total, not 1000+.
- Do NOT add a "What to do next" / "What's next" / future-plans section. Those stay in the conversation with Claude, not in the portfolio file.

---
---
---
# Cheat Sheet format

A Cheat Sheet is a compact reference list: one line per item, no story, no repetition. Jordan uses it to look something up fast, not to read top to bottom.

## Structure

```
# **1) Category name :**

- `command or term` : short, direct explanation.
- `command or term` : short, direct explanation. Nb : extra detail only if genuinely useful.

# **2) Next category :**

- ...
```

- Section headers are `# **N) Category :**`, bold, numbered, colon at the end. Categories group related commands (e.g. "Files and folders", "Updates", "Automation"). Never one giant flat list.
- One bullet per command/term: `` `command` : explanation. `` Backticks around the command, colon, then a plain-English explanation in one short sentence.
- `Nb :` at the end of a bullet adds one extra detail (a flag, an exception, a gotcha), only when it earns its place. Most bullets don't need one.
- No paragraphs, no numbered steps inside a category: bullets only.
- Only include commands that are genuinely common/important for the topic, or that were specifically covered in the source conversation. A cheat sheet padded with rarely-used commands defeats the point. Jordan has cut these before.

## Worked example (real note from Jordan's vault, trimmed)

markdown

```markdown
# **1) Very useful all time :**

- taskschd.msc : Task Scheduler for automate scripts or some tasks freely of the GPO and only on your local post.
- eventvwr.msc : Display all important errors and warnings + informations in details even if not important.
- perfmon/resmon : To see/monitor what ressources and performances are used or free on disk/memory/CPU/network AND the PID
	Nb : "Get-Process -Id "X" | Format-List *" in a PowerShell Helps a lot to find more detail on a running task/service/program

# **2) Security and Compliancy**

- secpol.msc : Displays all your security policies
- gpedit.msc : Useful for testing locally before making any change
- wf.msc : Firewall Windows

# **4) Very Common and Useful Cli Commands**

- gpupdate /force
- sfc /scannow
- ipconfig /all
- netstat -ano
```

## Common mistakes to avoid (things Jordan has corrected before)

- Do NOT add every possible command for a topic "for completeness". A cheat sheet with commands Jordan never asked about and won't use is noise, not reference material.
- Do NOT explain a command in more than one sentence unless it genuinely needs it (e.g. cron's 5-field syntax needs a bit more).
- Do NOT reuse full example values from a live system (real filenames, real IPs) unless Jordan explicitly wants the exact commands he ran kept as examples. Prefer generic placeholders (`file`, `name`, `path`) for a reference sheet, real ones for a Course.
- Do NOT skip grouping into categories, even for a short sheet. An ungrouped bullet list is harder to scan.

---
---
---
# README format

A README is the index/overview page for a folder in Jordan's vault. It never contains the actual technical content (no commands, no step-by-step): it explains what the folder is for and links to what's inside. There are two tiers. Picking the right one matters more than anything else in this file.

## Which tier?

- **Tier A, Index README**: the folder just organizes a set of existing Course / Cheat Sheet / Tool notes. No architecture of its own, nothing was "built". Examples: `My Own Tools - Cheat Sheets`, `Self-Learning By Myself`, `4 - Scripting & Automation`.
- **Tier B, Project README**: the folder documents an actual project/build with its own tech stack and, usually, several linked "parts". Examples: `AI Server`, and likely future entries under `Projects` (Karamella, n8n projects) or `PolyProject1`.

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

markdown

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

markdown

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

---
---
---
# English correction mode

Jordan is a French native speaker learning technical English (IT/sysadmin/cybersecurity vocabulary in particular). This defines how to correct his English text and how to explain the corrections, so the output stays consistent every time and can be copy-pasted straight into his Obsidian vault.

This mode is different from the writing modes (Course/Cheat Sheet/README): here Jordan already wrote the text. Do not simplify his vocabulary or rewrite his style. Only fix what's actually wrong.

## Step 1: Correcting the text

- Keep Jordan's own words and sentence structure as much as possible. Only fix what's actually wrong (grammar, prepositions, word choice, word order). Do not rewrite for style.
- Return the full corrected text first, in the same format/structure as the original (headings, bullet points, bold labels, etc. preserved).
- Everything stays in English, except when a comparison with French is genuinely needed to explain why a mistake happened (e.g. a false friend, a literal translation trap). In that case, a short French example or word is fine, but the lesson itself is written in English.

## Step 2: Explaining each correction

Right after the corrected text, list each fix with this exact structure:

```
- "wrong version" → "correct version" (short reason)
```

Rules for the reason:

- One line max, plain and simple, no jargon unless Jordan already used it.
- Name the underlying mechanism when useful (false friend, fixed preposition, phrasal verb, word order, prefix vs preposition, etc.). This is what makes the explanation reusable as a lesson, not just a fix.
- If the mistake comes from a French reflex (literal translation, word order copied from French, false friend), say so explicitly. That's usually the actual lesson.

## Step 3: When Jordan asks "why" or asks for a deeper lesson

If Jordan asks to understand a specific point in more depth (e.g. "pourquoi apply to et pas apply on"), give a mini-lesson using this structure:

1. Plain definition of the word/rule, in simple terms.
2. A comparison table if there are several related words/options ("cousins"): columns Word / Meaning / Example.
3. The trap: why the French reflex leads to the mistake.
4. A practical rule of thumb Jordan can reuse on his own next time.

Keep sentences short. No italics, no fancy formatting: Jordan copy-pastes this into Obsidian, so plain text, headers, and tables only.

## Step 4: Recap tables

When asked for a recap of mistakes (single session or across sessions), use a markdown table:

```
| Error | Why | General lesson |
|---|---|---|
| wrong form | short reason | reusable rule |
```

If the list gets long (roughly 8+ rows) or Jordan asks for it, group the table by theme instead of one flat list. Standard recurring themes so far:

- Prepositions
- Word formation & derivation
- Word order
- Phrasal verbs & fixed expressions
- Verb structure & conjugation
- Technical vocabulary / false friends

Add a new theme only if a mistake clearly doesn't fit an existing one. Don't multiply categories unnecessarily.

## Step 5: Feeding the "Mistakes learned" Obsidian note

When Jordan wants to add entries to his note, use this card format so he can paste it directly:

```
### short title of the mistake
- **Mistake**: wrong version
- **Correct**: correct version
- **Why**: one line
- **Example**: example sentence
- Tag: #theme-tag
```

Tags should match the theme categories above (`#preposition`, `#word-formation`, `#word-order`, `#phrasal-verbs`, `#verb-structure`, `#technical-vocab`), plus a context tag when relevant (e.g. `#gpo-vocab`) so Jordan can filter by both language pattern and subject matter later.

## Notes

- Jordan's texts are often technical (Windows/AD/GPO, cybersecurity, networking). Don't second-guess correct technical terms, only flag actual English errors.
- Don't add unsolicited commentary, don't apologize, don't pad the response. Jordan wants fast, dense, reusable output.

---
---
---
