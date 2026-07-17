# **1) The concept and the problem :**

I use Claude a lot now : in chat, in Claude Code, and in my automation stack (n8n -→ Mistral). I noticed something simple : the answer I get depends mostly on how I ask, not just on what I ask. Claude also has "Skills" now : small files (`SKILL.md`) that Claude can load by itself, so I don't have to repeat the same context every time. I wanted the simple rules for both : how to write a good prompt, and how to write a Skill that actually gets used.

# **2) How it's working ? (the basics) :**

- **Be clear** : say exactly what you want. Don't make Claude trying to know the missing part.
- **Give examples** : 1-2 examples of what you want (input → output) work better than a long explanation. (Personally I used 2-3 documents hand-made)
- **Separate the parts** : put your data, your instructions and your examples in clearly separated blocks (`<context>`, `<instructions>`...) so Claude doesn't mix them up.
- **Set the role once** : say who Claude should "be" and what the rules are, at the start, not in every message.
- **Ask for reasoning** : for anything with several steps or numbers, ask Claude to think step by step before answering. (Don't forget to tell explicitly to Claude or your AI "Explain to me all your thinking process step by step")
- **Cut big tasks into small ones** : one prompt per step is slower, but far more reliable than one giant prompt.
- **Say the format you want** : length, structure, tone. Don't leave it to guesswork. Ask it other parameter otherwise.
- **Ask Claude to check itself** : "before your final answer, check X, Y, Z" catches mistakes before they reach you. IMPORTANT TO SAVE TIME

# **3) Ready-to-use templates (detailed versions) :**

### Template A — General task, contract-style :

```text
Role : You are [role / expertise].
Objective : [the concrete outcome you need, one sentence].

Context :
[background, data, constraints, why this matters]

Rules :
1. [rule]
2. [rule]
3. [rule]

Edge cases :
- If [situation], do [X] instead of guessing.
- If information is missing, [ask / state the assumption / skip], don't invent it.

Example :
Input : [...]
Output : [...]

Success criteria : this is "done" when [concrete, checkable condition].
Output format : [exact structure — headers, length, code block, table, etc.]

Before answering, check your output against the rules and success criteria above.
```

### Template B — Turn messy notes into a clean documentation page (my note format) :

```text
Source : [paste raw notes / bullet points / half-finished draft]

Target : a short technical note, same structure as my usual ones :
1) The concept and the problem
2) How it's working
3) Concrete example / commands
4) What changed / improvements
5) How I set it up
6) Next steps

Style rules :
- Short sentences, simple words, no marketing tone.
- Keep every command / script line exact, don't paraphrase code.
- If something in my notes is unclear or incomplete, flag it instead of filling the gap yourself.
- Length : similar to my other notes, don't pad it.
```

To be fair, I prefer do it manually. Better comprehension and understanding.
### Template C — Debug a script (detailed diagnostic) :

```text
Script : [paste code/documents]
Environment : [OS, PowerShell/Python version, run as admin or not, interactive or Task Scheduler / cron]
Error / behavior observed : [exact error message or wrong output]
Expected behavior : [what it should do instead]

What I need, in this order :
1. Root cause in plain words — what line/logic is actually wrong, and why.
2. Minimal fix — change only what's broken, keep my naming and structure.
3. Why this fix won't break the rest of the script.
4. One thing I should test afterward to confirm it's really fixed.

Don't rewrite the whole script unless the whole logic is actually broken.
```

# **4) Skills : the upgrade :**

A prompt is one-shot, I write it every time. A Skill is written once, and Claude loads it by itself when it's useful.

- Only the **name** and a short **description** are always visible to Claude. The real instructions are only read once Claude decides the Skill is relevant.
- The description is what makes the Skill trigger or not. It has to say clearly what the Skill does and when to use it. Vague description = Skill never triggers.
- Give Claude freedom for open tasks. Give it an exact script for anything fragile or repetitive, so it can't improvise on something that shouldn't change.
- Keep it short. Everything inside a Skill takes space in Claude's memory once it's loaded.
- Test it for real : write it, use it on a real task, see what breaks, fix it. Don't just write it once and hope.

# **5) How to set one up (steps) :**

1. Choose where :
    - `~/.claude/skills/` → personal, works in every project
    - `.claude/skills/` (inside a repo) → shared with a team, saved in git
2. Create the folder and the file : `mkdir -p .claude/skills/my-skill-name`
3. Write `SKILL.md` :

```yaml
---
name: my-skill-name
description: [what it does, short]. Use when [what the user would actually say].
---

[Steps Claude should follow]
```

4. Use it :
    - Claude can trigger it by itself if the description matches
    - Or force it with `/my-skill-name` in Claude Code

# **6) Next steps :**

- Test any new Skill on a real task before trusting it.
- Turn recurring stuff (Windows docs, PowerShell rules, Sysmon config) into Skills instead of re-explaining every time.
- Use the templates above as a starting point, not a fixed rule.

# **7) Other detailed templates, for things I already do :**

### Log / alert triage (mairie monitoring stack : Vector → Loki → n8n → Mistral) :

```text
Alert : [paste raw log / alert content]
Source : [host, service, agent that raised it]
History : [has this asset triggered similar alerts before, how often]

Classify as : normal / suspicious / critical, using these rules :
- normal : known process, known user, known hours, no privilege change
- suspicious : unusual hour, unusual parent process, first time seen on this host
- critical : privilege escalation, known malicious pattern, lateral movement signs

Output :
1. Classification
2. Reasoning in 2-3 lines max
3. Confidence (low / medium / high)
4. Recommended next check
5. Only recommend an email alert if suspicious or critical — say clearly if it's a false positive
```

### KaramelIa — Suno / creative prompt (detailed) :

```text
I have mine and I kept it private
```

### Diplomatic / negotiation message (Consulat, or real life) :

```text
Role : You are a training instructor for new diplomats of [le Consulat / organization]. Skill to train : [de-escalation, alliance-building, information gathering, crisis response...] Trainee level : [beginner / intermediate / advanced] Build a training scenario with : 1. Situation brief (2-3 lines : who's involved, what's at stake, tension level) 2. Counterpart profile : personality, what they say they want, what they actually want (hidden agenda) 3. 3 choices the trainee can make, each with its likely consequence - Choice A : [firm stance] → likely result - Choice B : [conciliatory stance] → likely result - Choice C : [information-gathering / stalling] → likely result 4. Evaluation grid : what makes a strong response vs a weak one - Tone control - Information given away vs kept - Goal achieved vs conceded 5. Debrief questions to ask the trainee after they answer Difficulty lever (optional) : [add a hidden betrayal / time pressure / conflicting orders] Output format : one scenario card, ready to hand to a trainee, no more than one page.
```

### PowerShell script hardening (like the .NET auto-update script) :

```text
Script : [paste]
Run context : Task Scheduler, SYSTEM account, unattended most of the time

Check specifically for :
1. Admin rights verified before any install/change
2. Anti double-run protection (lock file or equivalent)
3. Clean logging : one line per action, timestamp, outcome
4. Error handling : exact command that failed, error message, timestamp saved for later
5. Reboot-needed flag tracked and reported at the end
6. No blocking prompts when run unattended (keypress only if launched interactively)
7. Alert only sent when something actually needs attention, not on every run

Tell me what's missing or weak, point by point. Don't rewrite what already works.
```

---

IMPORTANT : All this example are AI made, when I want a real skill I use my own skills to create it with the AI. But my work first.