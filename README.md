Skills for AI agents to interact with Cloud 66. Once skills are loaded, the agent will infer which skill to run based on your prompt.

# Claude
```
claude mcp add --transport http cloud66 https://app.cloud66.com/mcp

# add any skills you want by copying the relevant skill directory into .claude/skills
# for example, to install the `cloud66-deploy-application` skill, you will need to copy the `cloud66-deploy-application` directory to `.claude/skills/cloud66-deploy-application`

# inside claude, run /mcp and authenticate the Cloud 66 MCP server

# prompt the agent, e.g. "Let's create a Cloud 66 application"
```
