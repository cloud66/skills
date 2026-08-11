Skills for AI agents to interact with Cloud 66. Once skills are loaded, the agent will infer which skill to run based on your prompt.

## Available skills

| Skill | What it does |
| --- | --- |
| `cloud66-deploy-application` | Deploys an existing Cloud 66 application and follows the deployment to completion |
| `cloud66-create-csv3-application` | Creates a new CSv3 application on a cluster, from an image, a wizard, or the source code in the current directory |

## Connecting the Cloud 66 MCP server

Every skill needs the Cloud 66 MCP server at `https://app.cloud66.com/mcp`. It authenticates over
OAuth 2.1 with PKCE and registers clients dynamically, so there is nothing to create in the Cloud 66
dashboard beforehand — you just approve the permissions in your browser.

**Claude Code**

```
claude mcp add --transport http cloud66 https://app.cloud66.com/mcp

# inside claude, run /mcp and authenticate the Cloud 66 MCP server
```

**Claude Desktop**

Settings → Connectors → Add custom connector → URL `https://app.cloud66.com/mcp` → Connect.

**Other clients**

Add a remote (HTTP) MCP server pointing at `https://app.cloud66.com/mcp`.

Full instructions, including the discovery endpoints and what each permission means, are at
[help.cloud66.com](https://help.cloud66.com/rails/mcp/connecting-your-ai-agent).

## Installing a skill

Copy the skill directory you want into `.claude/skills`. For example, to install
`cloud66-deploy-application`, copy the `cloud66-deploy-application` directory to
`.claude/skills/cloud66-deploy-application`.

Then prompt the agent, e.g. *"Let's create a Cloud 66 application"*.

## Where the knowledge lives

These skills deliberately contain **workflow, not reference material**. Anything that could go stale
— tool arguments, scopes, cluster sizing rules, error meanings — is looked up at runtime from the
official documentation at [help.cloud66.com](https://help.cloud66.com), so the skills stay correct
as Cloud 66 changes.

Two things make this work for agents:

- **`https://help.cloud66.com/llms.txt`** is a curated index of every documentation page, grouped by
  category, linking directly to the Markdown version of each one.
- **Appending `.md` to any documentation URL** returns a clean Markdown version of that page, e.g.
  [`https://help.cloud66.com/rails/mcp/tools-reference.md`](https://help.cloud66.com/rails/mcp/tools-reference.md).

If you are writing a new skill, follow the same rule: describe the steps, and point at the docs for
the facts.
