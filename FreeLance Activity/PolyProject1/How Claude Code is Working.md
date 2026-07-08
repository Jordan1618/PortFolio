# **1) Why Claude Code :**

VibeCoding is a main part of project building especially when it came to understanding each layer on the project's stack and fundamental knowledge on how tools are functioning. 

So I have to understand this tool, that is the best in its  category (go look at main benchmarks publicly available). 

# **2) Claude Desktop / Web UI

Claude proposes to use different thing like commands or a local/ssh/shared folder or a authorisation granter. We can use /compact to compress  the memory and we can ask for a CLAUDE.MD to take a brief memory for a new conversation and saves tokens to daily/weekly limitation.

It can be used to create website/dashboard/automated local pentest tools/debug tools and etcetera. It's very impressive compared to the old time on how creating a project.

Claude can carry skills to like a Hermes agent. Some skills can be added via skills.sh (a giant bookshelf). For instance : https://www.skills.sh/coreyhaines31/marketingskills/copywriting for copywriting checks and claims.

Claude code can use connectors to automate some actions. It disposes few more advanced features : Under-agents, tests-tracking, remote control, apercu integrated, MCP connectors.

# **2.1) How skills are working ?**

Simply, skills are a .md with instructions oriented towards one or few missions. It's a file placed inside the agent folder. The usual command to add one is : npx skills add https://github.com/THEREQUESTEDSKILL --skill NAME

# **3) Claude Code like an engineer 

The source : https://www.youtube.com/watch?v=vvyIiacEl0I

The AI is stronger when iterating : 50% efficacity for the first and 5-10% per iteration.
We can use /loop /goal
Claude loop works on a 3-steps cycle : Reason(planing next step)-Act(implement,exec)-Look the result(Screenshot+test)
To put a good loop we ALWAYS need a right ending point : A metric to stop like when x = Y after n cycles
The Feedback can now be automated by another agent (for example we can have a fable 5 agent for feedbacking what's right or not and a opus 4.6 to code and works - so it's time and tokens saving)

# **3.1) How to make this theory works ?

5-8 iterations max (over, it's a waste of tokens and can add mistakes)
Importance of a great context : see skills and anthropic recommandations (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

In brief : 
1) Clarity and Maximum Specification
2) Motivation/Final Goal (Vocabulary Calibration)
3) For complex tasks (cut them into numerated steps)
4) Give an example of a waited output/right answer
5) Tells what's to do 


