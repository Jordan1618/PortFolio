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

Now covers 4 modes from one entry point (Step 0 routing): Course, Cheat Sheet, README (2 tiers), and English correction — each backed by its own reference file (`course-format.md`, `cheatsheet-format.md`, `readme-format.md`, `english-correction.md`). Hard constraints (English only, simple vocabulary, no invented commands/links, vault placement rules) apply to the 3 writing modes; correction mode runs on the opposite principle — keep Jordan's own wording, fix only actual errors.