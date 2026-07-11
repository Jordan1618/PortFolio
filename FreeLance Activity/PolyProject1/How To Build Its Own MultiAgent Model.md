# **1) How Use Claude Code for That (for a linux instance) :**

We can use Claude on a terminal, install it then asks it by %Claude of use it or VsCode or Desktop Gui
We create some .MD
We ask to Claude to init its Claude.MD (its root main file - To have the context and all memory)
Claude.MD contains : directories architecture, work rules, tones and styles asked, personas, MCP, Skills ...

1) Add files linked to your project
2) Tells to Claude "init a Claude.MD file"
3) Create or have a new .MD with a workflow or thinkflow like a procedure
4) Ask to Claude to transform it into a skill + the skill name + the need AND IMPORTANT : check the context with CLAUDE.MD and other folders + skill must follow the procedure + the tool to be use (model) + obligation to use "skill creator" to be skill-fitting
5) Create different directories according to your needs and fill them with your logicals and exemples
6) Connect MCP if needed and scoop correctly generated API keys
7) Agglomerate agents by naming them in a prompt and put an exit folder for every document susceptible to output. And "Make an agent team with X(or Y) agents of shared tasks like : 1) 2) 3)"

# **2) A Second Video about Claude Code :

https://www.youtube.com/watch?v=Wwkps2u4lp4&t=181s

1) He uses Claude on VCS and has a VPS on Hostinger for AI with 32Gb RAM
2) He inserts an ssh command and link its vsc with its vps
3) He opens its project in a terminal, then drag-and drops it into the code windows in vsc, it's looks like a ssh terminal
4) He checks the version with claude --version then claude agents (to open the agents system)

When I was trying, I Installed ClaudeCode on a PowerShell and it ends up to work. To be fair, I really enjoy the interface.
I needed to add :
$env:PATH += ";$env:USERPROFILE\.local\bin"
After /ide and validated my vsc.
I put a /bg to make a background working while I open a new powershell and used "claude agents" to access my targeted feature.

Useful commands available in vscode :
- /compact
- /clear (not the files cleared)
- /usage + /context to know
- /model --> /model opus
- /effort

# **3) Why a Second Brain ?


