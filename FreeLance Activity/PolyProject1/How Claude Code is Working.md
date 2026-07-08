# **1) Why Claude Code :**

VibeCoding is a main part of project building especially when it comes to understanding each layer of the project's stack and fundamental knowledge on how tools function. 

So I need to understand this tool, that is the best in its  category (go look at main benchmarks publicly available). 

# **2) Claude Desktop / Web UI

Claude proposes to use different things like commands or a local/ssh/shared folder or a authorization granter. We can use /compact to compress  the memory and we can ask for a CLAUDE.MD to keep a brief summary for new conversations and saves tokens against daily/weekly limitations.

It can be used to create website/dashboard/automated local pentest tools/debug tools and so on(=etc). It's very impressive compared to the old time on how creating a project.

Claude can carry skills just like a Hermes agent. Some skills can be added via skills.sh (a giant bookshelf). For instance : https://www.skills.sh/coreyhaines31/marketingskills/copywriting for copywriting checks and claims.

Claude code can use connectors to automate some actions. It features several advanced capabilities : Under-agents, test-tracking, remote control, integrated preview, MCP connectors.

# **2.1) How do skills work ?**

Simply, skills are a .md with instructions oriented towards one or few missions. It's a file placed inside the agent folder. The usual command to add one is : npx skills add https://github.com/THEREQUESTEDSKILL --skill NAME

# **3) Claude Code like an engineer 

The source : https://www.youtube.com/watch?v=vvyIiacEl0I

The AI is stronger when iterating : 50% efficacity for the first try and 5-10% per iteration after that.
We can use /loop /goal
Claude loop works on a 3-steps cycle : Reason(planing next step)-Act(implement,exec)-Look at the result(Screenshot+test)
To set up a good loop we ALWAYS need a clear endpoint : A metric to stop like when x = Y after n cycles
The Feedback can now be automated by another agent (for example we can have a fable 5 agent for feedbacking what's right or wrong and a opus 4.6 to code and works - saving both time and tokens saving)

# **3.1) How to make this theory work ?

5-8 iterations max (beyond that, it's a waste of tokens and can add mistakes)
Importance of a great context : see skills and anthropic recommandations (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

In brief : 
1) Clarity and Maximum Specification (flexible, if/else possibility)
2) Motivation/Final Goal (Vocabulary Calibration)
3) For complex tasks (cut them into numerated steps)
4) Give an example of a expected output/right answer
5) Tells it what to do instead of what's to not to do
6) Give a role
Use of XML marks like <instructions/>What you do</instructions/> (delete the final /) or <thinking/></thinking/> to put the AI thinking process inside
Common traps :
- Total blurriness
- Asking for several very different tasks
- Contradictions
- Hostility towards it
- Define too much (marks, json code, etc)
Professional tips : 
- Define success criteria
- One modification at a time to see where the problem is
- Use AI to upgrade your prompt by asking it
- Once it's working, delete redundant sentences to save tokens
More Explanations :
1) Claude uses a "just in time"context approach by keeping in mind light references like folder ways or links instead of full files.
2) For large context : Compact / Summary by AI like the CLAUDE.MD / Multi-agent synthesis
An exemple of correct prompt is :
	You are a customer support agent for Claude’s Bakery. You specialize in assisting customers with their orders and basic questions about the bakery. Use the tools available to you to resolve the issue efficiently and professionally.
	You have access to order management systems, product catalogs, and store policies. Your goal is to resolve issues quickly when possible. Start by understanding the complete situation before proposing solutions, ask follow-up questions if you do not understand.
	### Response Framework:
	1. **Identify the core issue** - Look beyond surface complaints to understand what the customer actually needs
	2. **Gather necessary context** - Use available tools to verify order details, check inventory, or review policies before responding
    3. **Provide clear resolution** - Offer concrete next steps with realistic timelines
    4. **Confirm satisfaction** - Ensure the customer understands the resolution and knows how to follow up if needed
    ### Guidelines:
	- When multiple solutions exist, choose the simplest one that fully addresses the issue
	- If a user mentions an order, check its status before suggesting next steps
	- When uncertain, call the human_assistance tool
	- For legal issues, health/allergy emergencies, or situations requiring financial adjustments beyond standard policies, call the human_assistance tool
	- Acknowledge frustration or urgency in the user’s tone and respond with appropriate empathy

Now come back to the video :





