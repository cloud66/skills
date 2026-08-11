---
name: cloud66-create-csv3-application
description: Create a Cloud 66 CSv3 application.
---

When creating a Cloud 66 application, always do the following.

# Prerequisites
1. The root Cloud 66 URL is `https://app.cloud66.com`, from now on referred to as `<BASE_URL>`.
2. Make sure the Cloud 66 MCP server is connected and authenticated — see the `Connecting the MCP server` section.
3. Follow the steps in the `Application creation` section.

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

- `https://help.cloud66.com/rails/mcp/tools-reference.md` — every tool, its arguments and its scope,
  including the full shape of `services`, `databases` and `environment_variables`
- `https://help.cloud66.com/deploy/getting-started/getting-started-with-clusters.md` — how clusters work
- `https://help.cloud66.com/deploy/getting-started/deploy-v2-vs-v3.md` — what CSv3 is and how it differs from v2
- `https://help.cloud66.com/rails/mcp/troubleshooting.md` — what an error means and how to fix it

If the documentation and this skill disagree, the documentation is correct.

# Application creation
These are the steps for creating an application, which is done using the `create_csv3_application`
tool:

1. Ask the user which cluster the new application should be deployed to. List the candidates with
   the `list_clusters` tool. The chosen cluster must have a `status` of `"success"` and a
   `platform_version` of `3.x`.
   - If there are no suitable clusters, offer the user two options: create one in the dashboard at
     [https://app.cloud66.com/clusters/new](https://app.cloud66.com/clusters/new), or create one
     here with the `create_csv3_cluster` tool. If they choose the tool, read the cluster section of
     `https://help.cloud66.com/rails/mcp/tools-reference.md` first — the node pool, availability and
     sizing rules are strict, and `list_cloud_regions` and `list_cloud_sizes` give you the valid
     region and size IDs for the chosen cloud.
2. Ask the user for the basics: the new application's name and its environment.
3. Populate the remaining fields for `create_csv3_application`. Ask the user to choose one of these
   three workflows:
   - Deploy a simple NGINX image — follow `Application creation via NGINX image`.
   - Follow a wizard to populate the remaining fields — follow `Application creation via wizard`.
   - Deploy the source code from the current directory — follow `Application creation via source code`.

# Application creation via NGINX image
1. Populate `services` with a single image-based service called `web`, running the `nginx:latest`
   image and exposing its internal port `80` externally to HTTP 80 and HTTPS 443. No other fields
   are required.
2. Follow the steps in the `Calling the tool` section.

# Application creation via wizard
1. Guide the user through populating `services` — ask what kind of service they want to deploy
   (backed by an image, or by a git repository), what the values should be, which ports should be
   exposed, what command runs the application in the container, and so on.
2. Ask whether these services need any additional environment variables, and populate
   `environment_variables` accordingly.
3. Ask whether these services need any databases. If so, ask whether to attach existing ones (found
   via the `list_cluster_databases` tool) or help the user define new ones.
4. Ask whether any service needs a database migration command (for example
   `bundle exec rails db:migrate` for Rails applications), and if so populate `migration_command`
   and `migration_service_name`.
5. Follow the steps in the `Calling the tool` section.

# Application creation via source code
1. You will be creating a single service based on the source code.
2. Determine the current upstream git repository URL for the service's `git_repo` field.
3. Determine the current git branch for `git_branch` and confirm with the user that this is the
   branch they want to deploy. If they want a different one, switch to it.
4. Analyse the codebase for existing Dockerfiles, which database types might be required
   (MySQL/Redis/PostgreSQL), and which environment variables are likely needed (for example, every
   Rails application needs `SECRET_KEY_BASE` set to a random value).
5. If the codebase already has Dockerfiles, ask the user which one to use for `dockerfile_path`, and
   what `build_root` should be.
6. If there are no Dockerfiles, generate one and refine it with the user until they are happy. Then,
   with their permission, commit it to this branch and push it upstream.
7. If the analysis suggested a database is needed, guide the user through creating a new one or
   choosing an existing one via the `list_cluster_databases` tool.
8. Ask whether the service needs a database migration command (for example
   `bundle exec rails db:migrate`), and if so populate `migration_command` and
   `migration_service_name`.
9. Suggest reasonable environment variables for this codebase, and ask what else the user wants to
   add — this populates `environment_variables`.
10. Follow the steps in the `Calling the tool` section.

# Calling the tool
Once you have all the parameters, call the `create_csv3_application` tool. It is *extremely
important* that this tool's nested parameters are passed as structured JSON objects and arrays,
**not** as JSON-encoded strings.

`create_csv3_application` is asynchronous: it returns an operation handle. Poll it with
`get_operation` until it finishes, then report the result to the user.

A few tools — `delete_application`, `delete_deployment_profile` and `create_cloud` — pause and ask
the user to confirm in their browser before they run. This is expected. Relay the prompt to the
user, wait, and do not attempt to work around it. See
`https://help.cloud66.com/rails/mcp/approving-sensitive-actions.md`.
