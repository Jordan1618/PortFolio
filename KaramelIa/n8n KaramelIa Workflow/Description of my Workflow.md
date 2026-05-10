# n8n KaramelIa Workflow : Automation Plan

## Purposes :
I am currently mapping out the logic to transform a single cinematic quote into a full set of production-ready assets (Audio, Video, and Metadata) without (or the less possible) manual intervention. I will work on it when i will have the time.

## Tech Stack & Planned Nodes :
- **Source**: Manually in the workflow.
- **Orchestrator**: n8n (Self-hosted on docker).
- **Processing**: Future API integrations for prompt expansion and asset generation.
- **Goal**: A "hands-off" workflow that prepares everything for the final CapCut edit.

## Method (Under Design) :
I am currently structuring the logic into a 3-step sequence:
1. **The Trigger**: Capturing the "Climax Quote" in a specific Obsidian folder.
2. **The Logic**: Using n8n to parse the text and distribute instructions to different AI models.
3. **The Gathering**: Automatically centralizing all generated files into a dedicated production folder for the weekly mix.

## Next Steps :
- [ ] Finalize the logic flow diagram in Obsidian Canvas.
- [ ] Set up the n8n environment and test the first Webhook triggers.
- [ ] Connect the first API (OpenAI/Claude) for automated prompt engineering.