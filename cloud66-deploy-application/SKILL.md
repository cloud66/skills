---
name: cloud66-deploy-application
description: Deploys a Cloud 66 application.
---

When deploying a Cloud 66 application, always do the following:

# Prerequisites
1. The root Cloud 66 URL is `https://app.cloud66.com`, from now on referred to as `<BASE_URL>`.
2. Make sure that the Cloud 66 MCP server `<BASE_URL>/mcp` has been added. If it has not, prompt the user to do so.
3. Make sure that the Cloud 66 MCP server has been authenticated against. If it has not, prompt the user to do so. This MCP server supports OAuth 2.0 Dynamic Client Registration. Make sure to request *all* scopes and let the user narrow down in the UI - the list of available scopes is available at `<BASE_URL>/.well-known/oauth-protected-resource/mcp` and `<BASE_URL>/.well-known/oauth-protected-resource` as per the MCP spec. Tell the user exactly what command to run for the specific agent they are running.
4. Follow the steps in the `Application deployment` section.

# Application deployment
4. Ask the user what the name of the application they want to deploy is.
5. Use a combination of the `list_accounts` and `list_applications` tools to determine the account ID and application ID that the user is requesting to deploy.
6. Once you have the account ID and application ID, use the `create_deployment` tool to enqueue a deployment of the application.
7. If the deployment has been successfully enqueued, tell the user that the logs are available by visiting `<BASE_URL>/operations/<DEPLOYMENT_ID>` - where `DEPLOYMENT_ID` is the ID of the deployment that was just enqueued.
8. If the deployment has been successfully enqueued, continuously poll the `get_operation` tool with exponential backoff and full jitter (minimum 10 seconds, maximum 60 seconds, full jitter means once you determine the current backoff value, choose a random value between 0 and that value) and wait until the deployment is done.
9. If the deployment was successful, there is nothing else to be done. If it has failed, call the `get_operation_logs` tool and analyze the logs to determine what the issue is. Summarize the issue to the user and offer helpful suggestions on how they could fix it.
