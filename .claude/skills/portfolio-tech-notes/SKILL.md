---
name: portfolio-tech-notes
description: Writes IT/sysadmin/scripting documentation for Jordan's Obsidian portfolio vault, in one of three fixed formats — "Course" (a step-by-step narrative note about something he just learned or debugged), "Cheat Sheet" (a short bullet-point reference list), or "README" (an index/overview page for a vault folder). Use this skill whenever Jordan asks to write up, document, or turn a conversation into a course, a cheat sheet, portfolio notes, a README, or vault notes about an IT topic — even if he just says "fais-moi un doc là-dessus", "note ça", "cheat sheet sur X", or similar, without naming the format explicitly. Also use to shorten, simplify, or reformat existing course/cheat sheet/README drafts to match this style, and to advise where a new note should live in his vault. Also corrects Jordan's English text (grammar, prepositions, word choice, word order), explains each fix, saves a lesson per mistake under "English Self-Learning" and logs it in Mistakes Learned and Vocabulary, whenever he asks to correct, check or proofread English text, or asks "why" about an English mistake.
---

# Portfolio Tech Notes

Generates Jordan's IT portfolio documentation in English, matching the exact tone and structure of his existing vault. He has corrected this format multiple times — treat the rules below as firm, not stylistic suggestions.

## Hard constraints (apply to the 3 writing formats, not to English correction)

- Always English, regardless of the language Jordan writes the request in.
- Simple, student-level vocabulary. Short sentences.
- Only real, necessary commands — never invent a command, flag, or link that wasn't actually used/confirmed.
- Written as Jordan's own notes: first person for Courses, neutral reference tone for Cheat Sheets and READMEs.
- Output as a single `.md` file, created with the file tools, in the folder you and Jordan agree on (see Step 4).

## Step 0 — figure out the format

- "Fais-moi un doc / note ça / documente ça" about something just done or debugged → **Course**.
- "Cheat sheet sur X / liste-moi les commandes" → **Cheat Sheet**.
- "Fais-moi le README de ce dossier / index cette section" → **README**.
- "Corrige mon anglais / corrige comme tout à l'heure / pourquoi apply to et pas apply on" → **English correction** (Part B). Skip Steps 1 to 4 below and follow `references/english-correction.md` instead, in full, in order.
- If genuinely ambiguous, ask Jordan rather than guessing.

## Step 1 — read the right reference file

- Course → `references/course-format.md`
- Cheat Sheet → `references/cheatsheet-format.md`
- README → `references/readme-format.md` (has 2 tiers — read the "Which tier?" section first)
- English correction → `references/english-correction.md`

Read the matching reference file in full before writing. Follow its structure exactly.

## Step 2 — gather the source material

- Don't invent commands or steps. Only use what was actually run/confirmed in the conversation or provided by Jordan.
- Keep only the final correct path — don't narrate every failed attempt unless the failed attempts are themselves the lesson (common in Course notes).
- If key details are missing (exact command, exact error, which folder this belongs in), ask Jordan instead of guessing.

## Step 3 — write it

- Follow the reference file's structure exactly (headers, code blocks, `Nb :` usage, etc.).
- Self-check against the hard constraints above before presenting the result.
- No intro paragraph, no closing "what's next" section (see reference files for details — this is a repeated correction Jordan has made).

## Step 4 — advise on placement in the vault

Vault folder map:

- `AI Server` — AI server project notes (Tier B README candidate)
- `English Self-Learning` — English study notes (written by the English correction mode, not by the 3 writing formats)
- `FreeLance Activity` — freelance/business notes
- `My Own Tools - Cheat Sheets` — tool docs and cheat sheets (Tier A README)
- `Pièces jointes` — attachments, not documentation
- `Projects` — project folders (KaramelIa, n8n, PolyProject1, etc. — usually Tier B README)
- `Russian Self-Learning` — Russian study notes
- `Self-Learning By Myself (0-6)` — general self-learning topics (Tier A README)
- `Skills & MasterPrompts` — skill documentation (this folder)

Routing rules:

- Match the topic to the closest existing folder. Don't invent a new top-level folder without asking.
- File naming: match the casing/style of existing files in that folder (spaces allowed, no forced kebab-case).
- A README always lives inside the folder it indexes, filename `README.md`.
- State the suggested folder + filename alongside the generated file, don't just create it silently.

## Notes

- Jordan's topics are usually Windows/AD/GPO, cybersecurity, networking, scripting.
- If Jordan wants an existing draft shortened/simplified/reformatted, apply the same format rules to the existing text instead of writing from scratch.
