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
