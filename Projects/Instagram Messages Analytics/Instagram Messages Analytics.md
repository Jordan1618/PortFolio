# **1) What I built :**

A local pipeline that turns my own Instagram data export into four French reports: a general review, a one-page summary, an "openers" report and a "who I am, how I changed" report over 9 years. It reads about 446,000 messages and 46,000 voice messages, transcribes the voice messages, measures how I communicate, sorts about 2,160 conversations by type of relationship, and ends with a list of old contacts worth writing to, each with a score out of 10. Everything runs on my laptop. Nothing is uploaded.

Nb : the reports contain none of the other people's messages. Their names appear only in one private section, with numbers only.

# **2) Concepts I used :**

- Data cleaning : the same messages appeared in 18 export folders, and the text had encoding errors
- Speech to text : Whisper is a model that turns audio into text, and it runs on my own PC
- Text mining : word lists (lexicons) and regular expressions to spot tone, topics and questions
- Statistics : a confidence interval (Wilson) shows how sure a percentage is; small samples, correlation vs cause, survivorship bias
- Classification : my own rules to sort a conversation (short contact, light or deep friendship, romantic)
- Scoring : my own formulas that give a note out of 10 for link, depth, emotions and ease of restart
- Privacy and ethics : local processing, masked quotes, no analysis of anything sensitive from when I was a minor, no guessing of gender from a first name
- Performance tuning : threads, CPU load and Windows power settings
- Prompt and skill writing for an AI coding assistant

# **3) Stack and tools :**

- Python 3.13, faster-whisper (Whisper "small", int8, CPU only), PyAV, NumPy
- HTML, CSS and JavaScript (marked.js) for the web reports
- PowerShell and `powercfg` for the process and the power settings
- VS Code and Claude Code, with two skills I wrote for the method
- Headless Chrome screenshots to check the layout
- Windows 11, Intel Core Ultra 5 laptop, no graphics card

# **4) How I did it, step by step :**

1. Inventory : I listed and counted every folder before reading any message.
2. Cleaning : I read the JSON files, fixed the encoding and removed duplicates.
3. Manifest : a file listing which voice messages to transcribe (last 4 years), with date and sender.
4. Transcription : about 21,000 voice messages, newest first, saved after each file so I can stop and resume. About 4,900 are left.
5. Measures : per year and per conversation type, reply delays, how conversations start, follow-ups, with confidence intervals.
6. Classification and scores : each two-person conversation gets a type, and old contacts get a score out of 10.
7. Reports : written from a template, assembled from parts, built as coloured web pages, checked with screenshots.
8. Corrections : I review the results, fix errors, and rerun everything with two commands.

# **5) Mistakes and how I fixed them :**

- First report said "no text, no dates". I had only searched the audio folders. A full search found 2,300 JSON files. Fix : inventory the whole tree first.
- System messages (reactions, calls, attachments) were counted as texts, so many figures were slightly wrong. Fix : correct the filter, rerun every analysis, update all four reports.
- I drew conclusions from one conversation. Fix : use all the data.
- Transcription failed on every file because of a library version clash. Fix : I wrote my own audio decoder.
- A model download froze. Fix : I used a smaller model already on the PC and ran offline.
- Speed dropped from x4 to x1 overnight because the laptop went to sleep. Fix : turn off sleep on power and keep the program awake.
- Voice messages looked richer in my word lists, only because they are longer. Fix : compare per 1,000 words.
- The model invented repeated phrases in 2.5% of voice messages. Fix : detect them and drop them.
- A "boundaries" metric was circular (it measured itself). Fix : remove it.
- My time estimates were wrong. Fix : estimate from a random sample, not from counts.

# **6) Potential :**

- Rerun in two commands on any new export
- Same method for WhatsApp or Telegram exports, customer support chats and call recordings
- Local transcription suits sensitive data (health, legal, HR) where cloud tools are not allowed
- Could become a small app with a dashboard

# **7) What it adds to my profile :**

- It moves me from sysadmin and scripting toward data and AI work, and fits my AI server notes
- It shows I can deliver from raw data to a readable report, with checks at each step
- It shows I can direct an AI assistant, verify what it produces and catch its mistakes
- It shows I handle privacy and ethics limits in a real data project

# **8) What employers can use :**

- Data analyst or data engineer : cleaning, deduplication, metrics, statistics
- AI and automation roles : local speech to text, long batch jobs that can resume
- Security and compliance : local processing of sensitive data, masking, documented limits
- Reporting : turning raw numbers into pages a non-technical person can read
