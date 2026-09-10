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

```markdown
# **1) What I wanted to do :**

I wanted to turn on Sysmon to log more details about what happens on my PC (which programs start, etc.), just to test what it changes. I found "Sysmon" as a checkbox in "Turn Windows features on or off" and ticked it. But it didn't work right away like a normal installed program.

Problems I got : Get-Service sysmon64 said "no service found with that name" Get-WinEvent on the Sysmon log said "no matching log found"

# **2) What was really happening :**

Ticking the checkbox only turns on the option. It does NOT start the program by itself, you still need one extra command. Also, I learned Windows 11 now comes with its own built-in copy of Sysmon (in `C:\Windows\System32\sysmon.exe`).

I installed the built-in one properly (PowerShell, as Administrator) :

```powershell
sysmon.exe -accepteula -i
```

Nb : `-accepteula` just skips the license popup. `-i` means "install".

# **3) Checking if it's worked :**

```powershell
Get-Service sysmon
```

Result : Status "Running". It works.

# **4) What I understood from reading the logs :**

- By default, Sysmon only logs two things : when a program starts (ID 1) and when it closes (ID 5)
- Raw logs are meant to be filtered first, not read one by one in a console
```

## Common mistakes to avoid (things Jordan has corrected before)

- Do NOT explain every command word-by-word if the command is simple (`ls`, `cd`, `cat`). Only break down commands with flags or syntax that isn't self-evident.
- Do NOT include commands that weren't actually used to solve the problem, even if they're "related" or "good practice".
- Do NOT add a generic intro like "In this note, I will show you how to...". Start directly at section 1.
- Do NOT use "you". This is Jordan's own log of what he did, not instructions to a reader.
- Keep it short: most of his real course notes are 300-500 words total, not 1000+.
- Do NOT add a "What to do next" / "What's next" / future-plans section. Those stay in the conversation with Claude, not in the portfolio file.
