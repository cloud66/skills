# Deploying a Cloud 66 application

Deploys an existing application and follows the deployment through to completion.

## Steps

1. Ask the user which application they want to deploy.
2. Call `list_accounts`, then `list_applications`, to resolve the account ID and application ID.
   Every tool except `list_accounts` needs an `account_id`.
3. Call `create_deployment` to enqueue the deployment.
   - If the user wants a specific branch, tag or commit, or a named deployment profile, check
     `https://help.cloud66.com/rails/mcp/tools-reference.md` for the exact argument names.
     `deployment_profile` is **mutually exclusive** with the individual deployment parameters.
   - `list_deployment_profiles` and `get_deployment_profile` resolve a profile by name.
4. Tell the user the logs are available at `https://app.cloud66.com/operations/<DEPLOYMENT_ID>`,
   using the ID of the deployment just enqueued.
5. Poll `get_operation` until the deployment finishes, using exponential backoff with full jitter —
   minimum 10 seconds, maximum 60 seconds. Once you have the current backoff value, sleep a random
   duration between 0 and that value.
6. If it succeeded, there is nothing else to do. Say so and stop.
7. If it failed, call `get_deployment_logs` and analyse the output to determine the cause. Summarise
   the problem and offer concrete suggestions. Before speculating about Cloud 66 behaviour, read
   `https://help.cloud66.com/rails/deployment/troubleshooting-deployments.md`.

## Related tools

`list_deployments` and `get_deployment` for history and detail; `list_servers` and `get_server` when
the failure looks server-side. Check the tool reference for their arguments.
