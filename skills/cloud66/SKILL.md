---
name: cloud66
description: Work with Cloud 66 — deploy applications, create CSv3 applications and clusters, inspect servers, databases, services and operations, and answer questions about how Cloud 66 works. Connects the Cloud 66 MCP server at https://app.cloud66.com/mcp and looks every fact up in the live documentation at help.cloud66.com and developers.cloud66.com. Use whenever the user mentions Cloud 66, c66, a Cloud 66 application, stack, cluster or CSv3, or asks to deploy or scale something on their own servers.
---

# Cloud 66

Cloud 66 builds, deploys and manages containerised applications on any cloud or on the user's own
servers. You drive it through the **Cloud 66 MCP server**, and you look everything else up in the
**live documentation**.

The root Cloud 66 URL is `https://app.cloud66.com`, referred to below as `<BASE_URL>`.

## Golden rules

1. **Never answer a Cloud 66 question from memory, and never guess a tool argument, a scope, a
   sizing rule or an error meaning.** Look it up first — see [Finding
   information](#finding-information). If the docs and this skill disagree, the docs are correct.
2. **Pass nested tool parameters as structured JSON objects and arrays, not as JSON-encoded
   strings.** Several Cloud 66 tools (`services`, `databases`, `environment_variables`, node pools)
   take nested values, and a stringified payload fails.
3. **Almost every tool needs an `account_id`.** `list_accounts` is the only exception — call it
   first, then resolve the rest.
4. **Never ask the user for a Cloud 66 password, API token or cloud provider credential.** The
   browser OAuth flow is the only supported way to grant access.
5. **Long-running tools are asynchronous.** They return an operation handle; poll `get_operation`
   until it finishes. Use exponential backoff with full jitter — minimum 10s, maximum 60s: once you
   have the current backoff value, sleep a random duration between 0 and that value.
6. **Some tools pause for browser confirmation** — including `delete_application`,
   `delete_deployment_profile` and `create_cloud`. That is expected and by design. Relay the prompt,
   wait, and never try to work around it. Details:
   `https://help.cloud66.com/rails/mcp/approving-sensitive-actions.md`.
7. **`cx` is not a substitute for the MCP server.** The `cx` Toolbelt is a separate CLI meant for the
   *user* to run in their own terminal. `cx login` only signs the CLI into the Cloud 66 web UI — it
   does not register this session's MCP connection and grants you no tools. If `cx` happens to
   already be on `PATH`, that changes nothing about Step 1 below: connect the MCP server yourself.

## Step 1 — make sure the MCP server is connected

The Cloud 66 tools (`list_accounts`, `list_applications`, `create_deployment`, …) come from the MCP
server at `<BASE_URL>/mcp`, and from nowhere else. It speaks Streamable HTTP and authenticates over
OAuth 2.1 with PKCE using Dynamic Client Registration, so there is nothing to pre-create in the
Cloud 66 dashboard. Do this before anything else in this skill, even if the user's request doesn't
mention it — do not substitute a `cx` command, the REST API, or anything else, and do not treat
`cx` being installed as equivalent to being connected.

If those tools are not available in this session, read
[`references/mcp-setup.md`](references/mcp-setup.md) and give the user the exact command for the
agent they are actually running. The short version for Claude Code:

```bash
claude mcp add --transport http cloud66 https://app.cloud66.com/mcp
claude mcp login cloud66
```

Run both yourself — `login` opens the user's browser for the OAuth consent screen and stores the
token once they approve, so there is nothing for the user to click inside Claude Code itself. Tell
them the browser is about to open and to approve access there. This can take a while (it's waiting on
a human), so don't treat a long-running `login` call as stuck.

Either way, the tools are still invisible to the *current* session — Claude Code only loads its MCP
list at startup, so `add` and `login` have no effect on the session that ran them. Tell the user to
exit and restart Claude Code now. Once they're back, the Cloud 66 tools are already authorized and
ready to use — no `/mcp` menu required.

## Step 2 — find the workflow

| The user wants to | Read |
| --- | --- |
| Deploy an existing application, and follow it to completion | [`references/deploying.md`](references/deploying.md) |
| Create a new CSv3 application — from an image, via a wizard, or from the code in this directory | [`references/creating-applications.md`](references/creating-applications.md) |
| Create or choose a cluster to deploy onto | [`references/creating-applications.md`](references/creating-applications.md) (cluster section) |
| Connect, authorize, re-authorize or troubleshoot the MCP server | [`references/mcp-setup.md`](references/mcp-setup.md) |
| Call the REST API directly — scripting, CI, or a client with no MCP support (not a substitute for Step 1) | [`references/rest-api.md`](references/rest-api.md) |
| Anything else — inspect servers, databases, services, backups, settings, or understand a product concept | Look it up (below), then use the matching MCP tool |

For anything not covered by a workflow file, the pattern is always the same: read the tool reference
for the exact arguments, call the tool, poll `get_operation` if it is asynchronous, and report back.

## Finding information

Both Cloud 66 documentation sites publish an [llms.txt](https://llmstxt.org) index and serve a clean
Markdown version of every page when you append `.md` to its URL. Always prefer the `.md` form.

**Product documentation — `https://help.cloud66.com`**

1. Fetch `https://help.cloud66.com/llms.txt` first. It is a curated index of every page, grouped by
   category (Getting Started, Build and Config, Databases, Deployment, Networking, Security,
   Servers, MCP Server, Toolbelt, and more), linking straight to each page's Markdown version.
2. Fetch the pages you need, e.g. `https://help.cloud66.com/rails/mcp/tools-reference.md`.
3. Links inside those Markdown files may contain the placeholders `/:product` and `/:version?`.
   Resolve them before fetching: replace `:product` with a product slug (`rails`, `deploy`, `sites`
   or `node`) and drop `:version?` unless you specifically need a Deploy version.

**API reference — `https://developers.cloud66.com`**

1. Fetch `https://developers.cloud66.com/llms.txt` for the index of the current (v3) API — Getting
   Started, Endpoints, Webhooks, Models and SDKs.
2. Fetch individual pages as Markdown, e.g.
   `https://developers.cloud66.com/v3/endpoints/deployments.md`.
3. `https://developers.cloud66.com/llms-full.txt` is every API page concatenated into one file
   (~300 KB). Use it only when you need to search across the whole API surface at once; otherwise
   fetch the individual pages, which is far cheaper.

The pages worth knowing by heart:

- `https://help.cloud66.com/rails/mcp/tools-reference.md` — **every MCP tool, its arguments, its
  required scope and its behaviour.** Read this before calling any tool you are not certain about.
- `https://help.cloud66.com/rails/mcp/troubleshooting.md` — what an MCP error means and how to fix it
- `https://help.cloud66.com/rails/mcp/permissions-and-scopes.md` — what each scope grants
- `https://help.cloud66.com/rails/mcp/approving-sensitive-actions.md` — the browser confirmation flow
- `https://help.cloud66.com/rails/deployment/troubleshooting-deployments.md` — diagnosing failed deployments
- `https://help.cloud66.com/deploy/getting-started/deploy-v2-vs-v3.md` — what CSv3 is
- `https://help.cloud66.com/deploy/getting-started/getting-started-with-clusters.md` — how clusters work

This skill deliberately contains **workflow, not reference material**. Anything that can go stale —
tool arguments, scopes, sizing rules, error meanings, API shapes — lives in the docs and is fetched
at runtime, so the skill stays correct as Cloud 66 changes.
