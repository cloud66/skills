# Connecting the Cloud 66 MCP server

Read this when the Cloud 66 tools are missing from the session, when authorization fails, or when
the user asks how to connect their agent.

## The endpoint

| Setting | Value |
| --- | --- |
| Server URL | `https://app.cloud66.com/mcp` |
| Transport | Streamable HTTP |
| Authorization | OAuth 2.1, authorization code + PKCE (`S256`), Dynamic Client Registration (RFC 7591) |

Because the server implements the standard remote-MCP discovery chain and registers clients
dynamically, there is **nothing to create in the Cloud 66 dashboard first**. The discovery endpoints
(protected-resource metadata, authorization server metadata, authorize/token/revoke/register) are
listed in `https://help.cloud66.com/rails/mcp/connecting-your-ai-agent.md` — fetch that page if a
client needs them spelled out.

## Adding it

Give the user the command for the agent they are *actually* running:

| Client | How to add it |
| --- | --- |
| Claude Code | `claude mcp add --transport http cloud66 https://app.cloud66.com/mcp`, then run `/mcp` and authenticate |
| Claude Desktop | Settings → Connectors → Add custom connector → name `Cloud 66`, URL `https://app.cloud66.com/mcp` → Add → Connect |
| Any other MCP client | Add a remote/HTTP MCP server with the URL `https://app.cloud66.com/mcp` |

## Authorizing

1. Tell the user to complete the consent screen that opens in their browser.
2. Ask for **all** scopes and let the user untick what the agent shouldn't have on that screen. The
   available scopes are published at
   `https://app.cloud66.com/.well-known/oauth-protected-resource/mcp`, per the MCP spec, and
   explained in `https://help.cloud66.com/rails/mcp/permissions-and-scopes.md`.
3. Never ask for a Cloud 66 password, API token or cloud provider credential. The browser flow is
   the only supported way to grant access.

Things worth telling the user when they come up:

- **Destructive tools need two-factor authentication** enabled on the Cloud 66 account. Read-only
  use does not.
- **Access tokens last 7 days.** When one expires the client re-runs the flow — the user approves
  once in the browser and carries on.
- **Re-authorizing the same client revokes its previous token.** Connecting the same agent on a
  second machine stops the first machine's connection until it reconnects.

## Checking it works

Cloud 66 organises everything under **accounts**, and every tool except `list_accounts` needs an
`account_id`. A good end-to-end check:

> "List my Cloud 66 accounts, then show me the applications in the first one."

## When it doesn't work

Read `https://help.cloud66.com/rails/mcp/troubleshooting.md` before speculating. It covers
connection failures, missing tools, permission errors and approval problems. Do not invent
workarounds — in particular, do not try to bypass a browser confirmation prompt.
