---
title: "Deploy Readiness"
weight: 80
---

You'll validate that the same Deep Agents support assistant can load as a LangGraph app, start the local LangGraph API server, and pass a health check. Hosted deployment in AWS LangSmith is optional for this workshop.

Why this matters: production deployment starts with a clean deployable surface. Before pushing anything to a hosted runtime, prove the graph config loads, the graph imports, the custom S3 store initializes, Gateway tools are discoverable, and the local API server can serve the app.

The `langgraph deploy` CLI path for the AWS LangSmith tenant is not the required path for this workshop. Use local deploy-readiness validation as the hands-on path. If your workspace has AWS LangSmith Deployment access, you can optionally deploy through the AWS LangSmith UI.

## The Deployable Graph

`langgraph.json` points at `graph.py:graph`. That module exports the same support-agent shape you built in the notebook: Gateway/MCP order and ticket tools, HITL-gated refunds, Bedrock KB lookup, AgentCore Browser research, support-reply guidance, and S3-backed `/memories/`.

```python
from deepagents import create_deep_agent

graph = asyncio.run(build_graph())
```

`langgraph.json` also registers the S3-backed store hook:

```json
{
  "graphs": {
    "support_tour": "./graph.py:graph"
  },
  "python_version": "3.12",
  "env": "./.env",
  "dependencies": ["."],
  "store": {
    "path": "./store.py:generate_store"
  }
}
```

Gateway, KB, and S3 environment values are required at import time. Run the Gateway registration script before validation if `.env` is missing stack outputs.

## Required: Local Deploy-Readiness Validation

Run from the repository directory that contains `langgraph.json`:

```bash
export LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com

aws sts get-caller-identity
uv run python scripts/register_gateway.py --write-env .env
uv run langgraph validate
uv run python -c "import graph; print('graph import ok:', type(graph.graph).__name__)"
uv run langgraph dev --no-browser --port 2024
```

In a second terminal, check the health endpoint:

```bash
curl -sS http://127.0.0.1:2024/ok
```

Expected:

```json
{"ok": true}
```

The required workshop path ends here. Stop the local server with `Ctrl-C` when you are done testing.

## Notebook-Friendly Validation

If you are running from the notebook, use the Part 7 validation cell. It runs the same checks, starts `langgraph dev` in the background, polls `/ok`, and prints the local Studio URL:

```text
https://aws.smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

Use the stop cell afterward to terminate the notebook-started server.

## Required-Path Gotchas

- Run from the directory with `langgraph.json`.
- If `aws sts get-caller-identity` returns `ExpiredToken`, refresh the same AWS credentials you used for the workshop stack.
- If `graph.py` fails to import, rerun `uv run python scripts/register_gateway.py --write-env .env` so Gateway, KB, and S3 values are present.
- If port `2024` is already in use, stop the old server or choose another port.
- Custom deployment stores are still an advanced path. This workshop registers the S3 store in `langgraph.json` through `store.py:generate_store`.

## Optional: Hosted Deploy in AWS LangSmith

Use this path only if your attendee account or facilitator workspace has AWS LangSmith Deployment access and a GitHub repo visible to the LangSmith GitHub app.

If you need AWS credentials for the hosted runtime, create an access key for the CDK-managed hosted deployment IAM user:

```bash
uv run python scripts/create_deployment_user_key.py --write-env .env
```

The script writes `AWS_REGION`, `AWS_DEFAULT_REGION`, `AWS_ACCESS_KEY_ID`, and `AWS_SECRET_ACCESS_KEY` to the local ignored `.env` file without printing the secret. Delete this key during cleanup.

In `https://aws.smith.langchain.com`:

1. Open **Deployments**.
2. Create a new deployment.
3. Select **Import from GitHub** and connect or authorize the GitHub app if prompted.
4. Select your fork or workshop repo.
5. Choose the branch to deploy.
6. Specify the full path to the LangGraph API config file, including the file name:
   - Standalone repo: `langgraph.json`
   - `langchain-aws-samples` monorepo: `examples/deepagents-aws-tour/langgraph.json`
7. Confirm the graph id is `support_tour`.
8. Add the required environment variables and secrets from `.env`.
9. Submit the deployment.
10. Copy the deployment API URL from the deployment details page.

Required deployment env vars/secrets:

```text
AWS_REGION
AWS_DEFAULT_REGION
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
BEDROCK_KB_ID
AGENT_FILES_BUCKET
PUBLIC_SUPPORT_DOC_KEY
GATEWAY_URL
COGNITO_TOKEN_URL
COGNITO_CLIENT_ID
COGNITO_CLIENT_SECRET
MCP_TRANSPORT
```

Do not commit `.env`, and do not add local-only values such as `DEPLOYMENT_URL`, `DEPLOYMENT_GRAPH`, `LANGSMITH_DEPLOYMENT_NAME`, or `LANGGRAPH_HOST_URL` to the hosted deployment.

After the UI deploy succeeds, set:

```bash
export DEPLOYMENT_URL=<deployment-api-url>
export DEPLOYMENT_GRAPH=support_tour
```

## Optional: Invoke the Hosted Agent

Pass a `customer_id` in run context so S3-backed `/memories/` is isolated per customer.

```python
import os
from langgraph_sdk import get_client

deployment_url = os.environ.get("DEPLOYMENT_URL")

if deployment_url:
    client = get_client(url=deployment_url, api_key=os.environ["LANGSMITH_API_KEY"])
    graph = os.environ.get("DEPLOYMENT_GRAPH", "support_tour")

    async for chunk in client.runs.stream(
        None,
        graph,
        input={"messages": [{"role": "human", "content": "SH-HUB-V2 wifi drops - known fix?"}]},
        context={"customer_id": "C-1042"},
        stream_mode="updates",
    ):
        if chunk.event == "metadata":
            print("run_id:", chunk.data.get("run_id"))
else:
    print("Set DEPLOYMENT_URL after an optional AWS LangSmith UI deploy to invoke the hosted agent.")
```

## Optional: Attach Feedback

In a product UI, a thumbs-up or thumbs-down button can call `create_feedback` on the deployed run.

```python
from langsmith import Client as LangSmithClient

if "run_id" in globals() and run_id:
    LangSmithClient().create_feedback(
        run_id=run_id,
        key="user_thumbs",
        score=1,
        comment="Workshop smoke-test feedback",
    )
    print("posted user_thumbs feedback to run:", run_id)
else:
    print("Run the optional hosted invocation first so run_id is available.")
```

Checkpoint: you should have validated the deployable graph locally with `langgraph validate`, imported `graph.py`, started `langgraph dev`, and confirmed `/ok` returns `{"ok": true}`. If you completed the optional hosted path, you should also have a deployment URL, a successful SDK invocation, and `user_thumbs` feedback visible on the trace.

Proceed to [Part 8 - Production Loop](/85-production-loop/).
