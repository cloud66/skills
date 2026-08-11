---
name: cloud66-deploy-application
description: Deploys a Cloud 66 application.
---

When deploying a Cloud 66 application, always do the following.

# Prerequisites
1. The root Cloud 66 URL is `https://app.cloud66.com`, from now on referred to as `<BASE_URL>`.
2. Make sure the Cloud 66 MCP server is connected and authenticated — see the `Connecting the MCP server` section.
3. Follow the steps in the `Application deployment` section.

# Connecting the MCP server
The Cloud 66 MCP server lives at `<BASE_URL>/mcp`. It speaks Streamable HTTP and authenticates over
OAuth 2.1 with PKCE, using Dynamic Client Registration — so there is nothing to pre-create in the
Cloud 66 dashboard.

If the Cloud 66 tools are not available, tell the user the exact command for the agent they are
actually running:

| Client | How to add it |
| --- | --- |
| Claude Code | `claude mcp add --transport http cloud66 https://app.cloud66.com/mcp`, then run `/mcp` and authenticate |
| Claude Desktop | Settings → Connectors → Add custom connector → URL `https://app.cloud66.com/mcp` → Connect |
| Other MCP clients | Add a remote/HTTP MCP server with the URL `https://app.cloud66.com/mcp` |

Then:
1. Tell the user to complete the browser consent screen. Ask for *all* scopes and let the user
   narrow them down there — the available scopes are published at
   `<BASE_URL>/.well-known/oauth-protected-resource/mcp` per the MCP spec.
2. Do not ask the user for a Cloud 66 password, API token, or cloud provider credentials. The
   browser flow is the only supported way to grant access.
3. For anything else — other clients, discovery endpoints, scope meanings, token lifetime, or
   connection errors — read
   `https://help.cloud66.com/rails/mcp/connecting-your-ai-agent.md` and
   `https://help.cloud66.com/rails/mcp/troubleshooting.md`.

# Finding information
Do not answer Cloud 66 questions from memory, and do not guess tool arguments or product behaviour.
Look them up on the official documentation site, `https://help.cloud66.com`:

1. Fetch `https://help.cloud66.com/llms.txt` first. It is a curated index of every documentation
   page, grouped by category, and it links straight to the Markdown version of each page.
2. Every page is available as clean Markdown by appending `.md` to its URL — for example
   `https://help.cloud66.com/rails/mcp/tools-reference.md`. Always prefer the `.md` form.
3. Links inside those Markdown files may contain the placeholders `/:product` and `/:version?`.
   Resolve them before fetching: replace `:product` with a product slug (`rails`, `deploy`, `sites`
   or `node`) and drop `:version?` unless you specifically need a Deploy version.

The most relevant pages for this skill:

- `https://help.cloud66.com/rails/mcp/tools-reference.md` — every tool, its arguments and its scope
- `https://help.cloud66.com/rails/mcp/troubleshooting.md` — what an error means and how to fix it
- `https://help.cloud66.com/rails/deployment/troubleshooting-deployments.md` — diagnosing failed deployments

If the documentation and this skill disagree, the documentation is correct.

# Application deployment
1. Ask the user which application they want to deploy.
2. Use `list_accounts` and then `list_applications` to resolve the account ID and application ID.
   Every tool except `list_accounts` needs an `account_id`.
3. Use the `create_deployment` tool to enqueue the deployment. If the user wants to deploy a
   specific branch, tag or commit, or to use a named deployment profile, check the tool reference
   above for the exact argument names — `deployment_profile` is mutually exclusive with the
   individual deployment parameters.
4. Once enqueued, tell the user the logs are available at `<BASE_URL>/operations/<DEPLOYMENT_ID>`,
   where `DEPLOYMENT_ID` is the ID of the deployment just enqueued.
5. Poll the `get_operation` tool until the deployment finishes, using exponential backoff with full
   jitter (minimum 10 seconds, maximum 60 seconds — once you have the current backoff value, sleep
   a random duration between 0 and that value).
6. If the deployment succeeded, there is nothing else to do.
7. If it failed, call `get_deployment_logs` and analyse the output to determine the cause. Summarise
   the problem for the user and offer concrete suggestions. Consult the documentation above before
   speculating about Cloud 66 behaviour.

# Calling tools
Some Cloud 66 tools take nested parameters. It is *extremely important* that these are passed as
structured JSON objects and arrays, **not** as JSON-encoded strings.

A few tools — `delete_application`, `delete_deployment_profile` and `create_cloud` — pause and ask
the user to confirm in their browser before they run. This is expected. Relay the prompt to the
user, wait, and do not attempt to work around it. See
`https://help.cloud66.com/rails/mcp/approving-sensitive-actions.md`.
