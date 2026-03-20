---
name: cloud66-create-csv3-application
description: Create a Cloud 66 CSv3 application.
---

When creating a Cloud 66 application, always do the following:

# Prerequisites
1. The root Cloud 66 URL is `https://app.cloud66.com`, from now on referred to as `<BASE_URL>`.
2. Make sure that the Cloud 66 MCP server `<BASE_URL>/mcp` has been added. If it has not, prompt the user to do so.
3. Make sure that the Cloud 66 MCP server has been authenticated against. If it has not, prompt the user to do so. This MCP server supports OAuth 2.0 Dynamic Client Registration. Make sure to request *all* scopes and let the user narrow down in the UI - the list of available scopes is available at `<BASE_URL>/.well-known/oauth-protected-resource/mcp` and `<BASE_URL>/.well-known/oauth-protected-resource` as per the MCP spec. Tell the user exactly what command to run for the specific agent they are running.
4. Follow the steps in the `Application creation` section.

# Application creation
These are the actual steps for creating an application, which is done using the `create_csv3_application` tool:
1. Ask the user which cluster they want this new application to be deployed to. The list of available clusters can be determined via the `list_clusters` tool. The selected cluster must have a `status` of `"success"` and `platform_version` of `3.x`. If there are no clusters, tell the user to create one first by vising [https://app.cloud66.com/clusters/new](https://app.cloud66.com/clusters/new).
2. Ask the user for some basic values: what the new application name is, and what the environment for it should be.
3. We now need to populate the rest of the fields for the `create_csv3_application` tool. Ask the user to choose one of these three workflows:
   - Deploy a simple NGINX image - follow the steps in the `Application creation via NGINX image`.
   - Follow a wizard to populate the remaining fields - follow the steps in the `Application creation via wizard`.
   - Deploy the source code from the current directory - follow the steps in the `Application creation via source code`.

# Application creation via NGINX image
1. Populate the `services` field with a single image-based service called `web` running the `nginx:latest` image and exposing its internal port `80` externally to HTTP 80 and HTTPS 443. No other fields are required.
2. Follow the steps in the `Calling the tool` section.

# Application creation via wizard
1. Guide the user to populate the `services` field - ask what kind of service they wish to deploy (backed by image, or git repository), what the values should be, what ports should be exposed, what the command to run the application in the container should be, e.t.c.
2. Ask the user if these services need any additional environment variables - populate the `environment_variables` field based on their answer.
3. Ask the user if these services need any databases, and if so, ask them if they want to use existing databases (seen via the `list_cluster_databases` tool), or help them populate the fields for creating a new database.
4. Ask if there are any database migration commands that need to be run for any services (e.g. `bundle exec rails db:migrate` for Rails applications), in which case the `migration_command` and `migration_service_name` fields can be populated.
5. Follow the steps in the `Calling the tool` section.

# Application creation via source code
1. We will be creating a single service based on the source code.
2. Determine the current upstream git repository URL for the `git_repo` field of `services`.
3. Determine the current git branch for the `git_branch` field of `services` and confirm with the user that this is the branch they want to deploy. If they want a different branch, switch to that branch.
4. Analyze the codebase for: existing Dockerfiles; which databases types might be required (MySQL/Redis/PostgreSQL); and what some necessary environment variables might be (for example, all Rails applications need the `SECRET_KEY_BASE` to be defined to be a random value).
5. If there are already Dockerfiles in the codebase, ask the user which Dockerfile should be used for the `dockerfile_path` field, and also what the `build_root` should be.
6. If there are no Dockerfiles in the codebase, generate one and work with the user to refine it until they are happy. Once they are happy, commit it to this branch and push it upstream (with the user's permission).
7. Guide the user through creating a new MySQL/Redis/PostgreSQL database or choose an existing one via the `list_databases` tool - if one was deemed necessary based on the analysis.
8. Ask if there are any database migration commands that need to be run for the service (e.g. `bundle exec rails db:migrate` for Rails applications), in which case the `migration_command` and `migration_service_name` fields can be populated.
9. Suggest some reasonable environment variables for this codebase to the user, and ask them what other ones they would like to add - this is for the `environment_variables` field.
10. Follow the steps in the `Calling the tool` section.

# Calling the tool
Once you have all the parameters, you can call the `create_csv3_application` tool. It is *extremely important* that this tool has nested parameters, which MUST be passed as structured JSON objects/arrays, NOT as JSON-encoded strings.
