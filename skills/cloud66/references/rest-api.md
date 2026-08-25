# Using the Cloud 66 REST API directly

The MCP server is the preferred way to work with Cloud 66, and is what gives you any Cloud 66 tools
in this session at all — see [Step 1](../SKILL.md#step-1--make-sure-the-mcp-server-is-connected) in
the main skill file. A session that simply hasn't connected the MCP server yet is not a reason to
fall back here — connect it first. Reach for the REST API only when the MCP server genuinely cannot
be used — the user is scripting something, integrating from CI, or working in a client with no MCP
support — or when they explicitly ask about the API.

## What to read

The API reference is at `https://developers.cloud66.com`, and every page is available as clean
Markdown by appending `.md`.

1. `https://developers.cloud66.com/llms.txt` — the index of the current (v3) API: Getting Started,
   Endpoints, Webhooks, Models, SDKs.
2. Individual pages, e.g.:
   - `https://developers.cloud66.com/v3/getting-started/authentication.md` — OAuth 2.0 and how to
     get a token
   - `https://developers.cloud66.com/v3/getting-started/scoped-access.md` — the scopes
   - `https://developers.cloud66.com/v3/getting-started/synchronous-vs-asynchronous.md` — how
     long-running calls behave
   - `https://developers.cloud66.com/v3/getting-started/http-response-statuses.md` — status codes
   - `https://developers.cloud66.com/v3/endpoints/<resource>.md` — one page per resource
     (`applications` live under `stacks`; also `deployments`, `servers`, `clusters`, `databases`,
     `services`, `jobs`, `backups`, …)
   - `https://developers.cloud66.com/v3/models/<model>.md` — the shape of each returned object
   - `https://developers.cloud66.com/v3/sdks/sdks.md` — official client libraries
3. `https://developers.cloud66.com/llms-full.txt` — every API page in one ~300 KB file. Use it only
   when you genuinely need to search the whole surface at once; individual pages are far cheaper.

## What to keep in mind

- The API is REST over JSON, and returns JSON for errors too.
- Authentication is **OAuth 2.0 with scoped access** — read the authentication page rather than
  assuming a header format, and never ask the user to paste a token into the conversation.
- Rate limit: **3,000 requests per user per hour**. Over that, requests return `503: throttled`
  until the next hourly interval.
- Many endpoints key off a **Stack UID**, an alphanumeric identifier for an application. List stacks
  to find it.
- Dates are ISO 8601 in UTC, e.g. `2014-06-15T14:13:18+00:00`.
- Long-running calls are asynchronous — read the synchronous-vs-asynchronous page for how to follow
  them to completion.

## The Toolbelt

For command-line work there is also the `cx` Toolbelt — a separate CLI for the **user** to run
directly in their own terminal, not something you should invoke on their behalf. `cx login` only
signs the CLI itself into the Cloud 66 web UI; it does not register this session's MCP connection
and grants you no tools. If you find `cx` already installed, that is not a substitute for Step 1 —
connect the MCP server as normal. Its documentation lives on the help site:
`https://help.cloud66.com/rails/toolbelt/using-cloud66-toolbelt.md` and
`https://help.cloud66.com/rails/toolbelt/commands.md`.
