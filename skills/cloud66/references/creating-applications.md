# Creating a CSv3 application

Creates a new application on a Container Stack v3 cluster with the `create_csv3_application` tool.

Before you populate anything, read the `create_csv3_application` section of
`https://help.cloud66.com/rails/mcp/tools-reference.md` — it has the full shape of `services`,
`databases` and `environment_variables`. If the user is unsure what CSv3 is, point them at
`https://help.cloud66.com/deploy/getting-started/deploy-v2-vs-v3.md`.

## 1. Choose a cluster

1. List the candidates with `list_clusters`. A usable cluster has `status` of `"success"` and a
   `platform_version` of `3.x`.
2. If there is no suitable cluster, offer the user two options:
   - Create one in the dashboard at `https://app.cloud66.com/clusters/new`, or
   - Create one here with `create_csv3_cluster`.

   If they choose the tool, read the cluster section of the tool reference **first**. The node pool,
   availability and sizing rules are strict — for example, `reduced_availability` allows exactly one
   worker pool and no manager pool, while `high_availability` requires manager counts of 3, 5 or 7
   and minimum worker sizes. Use `list_cloud_regions` and `list_cloud_sizes` to get the valid region
   and size IDs for the chosen cloud, and check the current rules in the docs rather than trusting
   this summary. Background: `https://help.cloud66.com/deploy/getting-started/getting-started-with-clusters.md`.

## 2. Ask for the basics

The new application's **name** and its **environment**.

## 3. Pick a workflow

Ask the user which of these three they want, then follow that section:

- **NGINX image** — the quickest way to get something running.
- **Wizard** — you ask questions and fill the fields in.
- **Source code** — deploy the code in the current directory.

### Via NGINX image

Populate `services` with a single image-based service called `web`, running the `nginx:latest` image
and exposing its internal port `80` externally to HTTP 80 and HTTPS 443. Nothing else is required.
Then go to *Calling the tool*.

### Via wizard

1. Guide the user through `services` — image-backed or git-repository-backed, the values for each
   field, which ports are exposed, the command that runs the application in the container, and so on.
2. Ask whether the services need extra environment variables, and populate `environment_variables`.
3. Ask whether they need databases. If so, ask whether to attach existing ones (found with
   `list_cluster_databases`) or to define new ones.
4. Ask whether any service needs a database migration command (for example
   `bundle exec rails db:migrate`). If so, populate `migration_command` and `migration_service_name`.
5. Go to *Calling the tool*.

### Via source code

1. You are creating a **single service** based on the source in the current directory.
2. Determine the current upstream git repository URL for `git_repo`.
3. Determine the current git branch for `git_branch`, and confirm it is the branch the user wants to
   deploy. If not, switch to the one they want.
4. Analyse the codebase: existing Dockerfiles, which database types are needed (MySQL, PostgreSQL,
   Redis), and which environment variables are likely required — for example every Rails application
   needs `SECRET_KEY_BASE` set to a random value.
5. If there are Dockerfiles already, ask which one to use for `dockerfile_path`, and what
   `build_root` should be.
6. If there are none, generate one and refine it with the user until they are happy. Then, **with
   their permission**, commit it to this branch and push it upstream.
7. If the analysis suggested a database, guide the user through creating a new one or choosing an
   existing one via `list_cluster_databases`.
8. Ask whether the service needs a migration command, and if so populate `migration_command` and
   `migration_service_name`.
9. Suggest sensible environment variables for this codebase and ask what else they want to add —
   this populates `environment_variables`.
10. Go to *Calling the tool*.

## Calling the tool

Call `create_csv3_application` with the parameters you have gathered. It is **extremely important**
that nested parameters are passed as structured JSON objects and arrays, **not** as JSON-encoded
strings.

`create_csv3_application` is asynchronous: it returns an operation handle. Poll `get_operation`
until it finishes — exponential backoff with full jitter, minimum 10s, maximum 60s — then report the
result to the user.

If the user then wants to deploy the new application, follow [`deploying.md`](deploying.md).
