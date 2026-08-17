# Cloud 66 agent skill

A single skill that teaches AI coding agents how to work with Cloud 66 — deploying applications,
creating CSv3 applications and clusters, inspecting servers, databases and operations, and answering
questions about how Cloud 66 works.

[![skills.sh](https://skills.sh/b/cloud66/skills)](https://skills.sh/cloud66/skills)

## Getting started

### 1. Install the skill

```bash
npx skills add cloud66/skills
```

That is the whole installation. The [`skills` CLI](https://skills.sh) works out which coding agents
you have and installs into each of them — Claude Code, Codex, Cursor, Copilot, OpenCode, Gemini CLI
and [70+ others](https://github.com/vercel-labs/skills#supported-agents). Nothing needs to be
registered or published first; the CLI reads this repository directly.

| Want to | Command |
| --- | --- |
| Install for every project, not just this one | `npx skills add cloud66/skills -g` |
| Install for one specific agent | `npx skills add cloud66/skills -a claude-code` |
| See what's in here without installing | `npx skills add cloud66/skills --list` |
| Update to the latest version later | `npx skills update cloud66` |
| Remove it | `npx skills remove cloud66` |

Prefer to do it by hand? Copy `skills/cloud66` into your agent's skills directory — for Claude Code
that is `.claude/skills/cloud66`, or `~/.claude/skills/cloud66` to make it global.

### 2. Just ask

```
Deploy my Cloud 66 application
```

The agent picks the skill up from your prompt — there is no command to run and nothing to enable.
Other things that will trigger it: *"create a Cloud 66 app from this repo"*, *"which of my Cloud 66
servers are unhealthy?"*, *"why did my last deployment fail?"*

### 3. Let it connect Cloud 66 for you

The skill drives Cloud 66 through the **Cloud 66 MCP server** at `https://app.cloud66.com/mcp`. You
do not have to set this up in advance: the first time the agent needs a Cloud 66 tool and can't find
one, the skill tells it how to add and authorize the server for the client you are actually running,
and it will walk you through it.

Authorization is OAuth 2.1 with PKCE and Dynamic Client Registration, so there is nothing to create
in the Cloud 66 dashboard beforehand — you approve the permissions in your browser, ticking off
anything the agent shouldn't have. You will never be asked for a password, API token or cloud
credential.

If you would rather add it yourself first, for Claude Code that is:

```bash
claude mcp add --transport http cloud66 https://app.cloud66.com/mcp

# then, inside Claude Code, run /mcp and authenticate
```

For Claude Desktop: Settings → Connectors → Add custom connector → URL `https://app.cloud66.com/mcp`
→ Connect. For any other client, add a remote (HTTP) MCP server with that same URL.

Full instructions, including the discovery endpoints and what each permission means, are at
[help.cloud66.com](https://help.cloud66.com/rails/mcp/connecting-your-ai-agent).

## Where the knowledge lives

The skill deliberately contains **workflow, not reference material**. Anything that could go stale —
tool arguments, scopes, cluster sizing rules, API shapes, error meanings — is looked up at runtime
from the official documentation, so the skill stays correct as Cloud 66 changes and does not need
updating with every release.

Two things make this work for agents:

- **`llms.txt` indexes.** [`help.cloud66.com/llms.txt`](https://help.cloud66.com/llms.txt) indexes
  every product documentation page, grouped by category;
  [`developers.cloud66.com/llms.txt`](https://developers.cloud66.com/llms.txt) does the same for the
  v3 API reference, with [`llms-full.txt`](https://developers.cloud66.com/llms-full.txt) carrying
  the whole API in one file.
- **Appending `.md` to any documentation URL** returns a clean Markdown version of that page, e.g.
  [`help.cloud66.com/rails/mcp/tools-reference.md`](https://help.cloud66.com/rails/mcp/tools-reference.md)
  or [`developers.cloud66.com/v3/endpoints/deployments.md`](https://developers.cloud66.com/v3/endpoints/deployments.md).

## Layout

```
skills/cloud66/
├── SKILL.md                              # golden rules, MCP check, doc lookup, routing
└── references/
    ├── mcp-setup.md                      # connecting, authorizing, troubleshooting
    ├── deploying.md                      # deploy an application and follow it
    ├── creating-applications.md          # create a CSv3 cluster and application
    └── rest-api.md                       # the REST API and Toolbelt, when MCP isn't an option
```

`SKILL.md` is what the agent always loads; the `references/` files are read on demand for the task
at hand. If you extend this, follow the same rule: describe the steps, and point at the docs for the
facts.
