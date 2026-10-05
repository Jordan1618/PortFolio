# **1) What I wanted to do :**

Writing fifteen long guides with an AI gives a different style and different mistakes at each session. I wanted an optimized Claude: one that writes the guides the same way every time and does not forget the boring steps (sources, indexes, links). Over two months this became a small system of files, each one with a precise job. This note explains the system and why each file lives where it lives.

# **2) The files, and why each one is where it is :**

| File | Where | Loaded when | Why there |
|---|---|---|---|
| `CLAUDE.md` | root of the repository | automatically, at the start of every Claude Code session in this folder | It guarantees the most critical rules are applied even if nobody thinks to reread the long documents. It is short on purpose |
| `MAINTENANCE.md` | root of the repository | when I or Claude read it | The long reference: the rules and the why behind them (structure, frontmatter, links, checklist after each addition) |
| `.claude/skills/` | inside the repository | on demand, when a skill is called | A project skill is versioned with the code. Anyone who clones the project gets the method. The personal version stays in Claude Desktop |
| `5 - Notes Internes/` | inside the repository, not published on the site | when needed | Maintainer material (audits, worksites, request log) must never reach readers, but must survive a change of machine or session |
| `Historique des demandes.md` | in `5 - Notes Internes/` | read when resuming | A table of every request I sent (number, date, time, context in one sentence, the request itself), so another Claude session can understand how the project got here |
| Chantier files | in `5 - Notes Internes/` | read when resuming a big task | The original request reformulated for someone who does not have the conversation, the rules, a state table per sub-task, and how to resume |

The difference between the three layers is the key idea. `CLAUDE.md` is the reminder that is always read, `MAINTENANCE.md` is the manual, and a skill is a method that is called only when needed. Putting everything in one file would make it too long, and a long file is a file that is ignored.

Nb : the skill also exists at user level, in my Claude Desktop, under the name "guide-compagnon-attentif". It is written to me by name, with trigger phrases ("make me a complete guide on X", "triple the length"). The project copy "Faiseur2Guide" is generic ("we" instead of my name) and has a short description, because it travels with the repository.

# **3) The four skills :**

- **Faiseur2Guide** (the planner, v19): personas by subject, the grid of 100 angles in 10 families, the inventory of the repository, the overlap question, two examples of 30 sub-topics shown in the same message as the grid. It stops at the plan of chapters. The full history is in the [skill doc](../../Tools%20%26%20Skills/Skills%20%26%20MasterPrompts/Skills/Faiseur2Guide.md).
- **Redaction2Chapitre** (the writer): one thread from top to bottom, define the object (not only its parts), one analogy carried from the introduction to the end and turned around at the end, write the sentence first then source it, explain every study (what the researchers did, what came out, why it is clever), give every figure its scale, and a floor of 1,500 words. Two modes: deep (full rewrite, two research rounds) and surgery (keep what holds, add what is missing).
- **Audit2Guide** (the reader, read-only): a nine-point grid per chapter and three verdicts (nothing to do, surgery, rewrite). It never corrects anything, because an audit that fixes things on the way makes its own report wrong.
- **Liste2Guides** (the recommender, first called RecommandeurDeGuides): proposes new guides with the same elicitation method, 160 suggestions, each labelled core or peripheral but never discarded.

The blocks that make the rules checkable:
- **⚖️ Nuance:** one per chapter, it lists the misunderstandings about a key term ("openness is not intelligence") and what each confusion costs.
- **👁️ Seen from the other side:** only where a perception gap is documented.
- **💑 In the couple:** what the subject changes for two people.
- **🗣️ Real testimony:** a published testimony with a link, never invented.

# **4) The five levers, a technique the skill enforces :**

An open emotional question ("how do you feel?") blocks more than it opens, because it asks for an answer the person does not have at hand. When a chapter proposes a sentence to say out loud, it must use one of five levers: a closed menu, the body (where, not what), a number (out of ten), a before/after difference, or "when" instead of "why". Example from the guide on questions: "how are you?" becomes "is it more tiredness, anger, or something else?", or "on ten, where are you right now?".

# **5) Problems I met :**

- **Rules ignored under pressure.** The first skill was about 250 lines and only six lines were about writing. The rules about analogies, testimonies, "seen from the other side" and chapter length were ignored on 32 chapters in a row, because only checkable rules (is the source in the folder) survive when the volume is high. The answer: split the writing method into its own short skill, and turn each craft rule into a block that can be counted.
- **Elicitation skipped.** Several times Claude started writing before asking the questions, or did not show the two examples of 30 sub-topics. I hardened the rule in `CLAUDE.md`: the elicitation is done in the conversation, never delegated to a background agent, and nothing is written before I validate.
- **Invented references and testimonies.** The rule "never fabricate" and the rule "say it when nothing is found" were written into the skill (see [Sourcing Method](Sourcing%20Method.md)).
- **A dashboard that measured the wrong thing.** The follow-up file only tracked the word count, so the guides grew without becoming better.
- **Session limits and tokens.** Mass audits hit a session cap. I learned to use fewer, wider agents (an audit agent in read-only mode costs less context because the reading never goes back to the main window), to leave mechanical work to scripts, and to check `git status` before each commit so one agent does not overwrite another.
- **Work that is not deployed.** Claude worked on a branch and I saw no change on the site, because only `main` is deployed. Always check which branch the site is built from, and tell Claude the destination branch.
- **Losing the thread between sessions.** The portable chantier files and the request log fix this.

# **6) What I learned :**

- A good prompt for a long project is a set of rules born from real failures, not a long description
- AI is fast at drafting and weak at checking itself, so the checks must be written as steps and, when possible, as scripts
- Keep a skill short: a long skill is a skill that is ignored
- Separate who speaks (persona), how the work is organized (method), how a chapter is written (craft), and what is forbidden (rules)
- Document for the next session: another Claude, on another machine, must be able to continue without the conversation
