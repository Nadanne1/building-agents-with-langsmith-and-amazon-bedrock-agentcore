---
title: "Deploy"
weight: 80
---


**You'll build:** Deploy the same agent to LangSmith Deployment via `langgraph deploy`. Invoke the deployed agent over the LangGraph SDK and attach user feedback to a production trace.

**Why this matters:** Local agents are fine for development. Production needs a managed runtime - process supervision, scaling, deployment versioning, integration with observability. Because DeepAgents compiles to a standard LangGraph graph, there's nothing special to do beyond ordinary LangGraph packaging.

---

## The Deployable Graph

`graph.py` exports the agent with Gateway/MCP tools, HITL-gated refunds, KB lookup, Browser research, and S3-backed `/memories/`:

```python
from deepagents import create_deep_agent
graph = build_graph()
```

`langgraph.json` points at it:

```json
{
  "graphs": {"support_tour": "./graph.py:graph"},
  "python_version": "3.11",
  "env": "./.env",
  "dependencies": ["."],
  "store": {"path": "./store.py:generate_store"}
}
```

---

## Deploy

Run from a terminal (not a notebook cell):

```bash
uv run langgraph validate
uv run langgraph deploy --name deepagents-aws-tour
```


If Docker is unavailable and your workspace supports remote builds, use `--remote`. If deploy returns `403 Forbidden`, your LangSmith key or workspace lacks Deployment access.


---

## Invoke the Deployed Agent

```python
from langgraph_sdk import get_client

deployment_url = os.environ["DEPLOYMENT_URL"]
client = get_client(url=deployment_url, api_key=os.environ["LANGSMITH_API_KEY"])

async for chunk in client.runs.stream(None, "support_tour",
        input={"messages": [{"role": "human", "content": "SH-HUB-V2 wifi drops - known fix?"}]},
        context={"customer_id": "C-1042"},
        stream_mode="updates"):
    if chunk.event == "metadata":
        print("run_id:", chunk.data.get("run_id"))
```

---

## Attach Feedback

Wire thumbs-up/down to the deployed trace:

```python
from langsmith import Client as LangSmithClient

LangSmithClient().create_feedback(
    run_id=run_id,
    key="user_thumbs",
    score=1,
    comment="Workshop smoke-test feedback",
)
```

In a product UI, this same `create_feedback` call sits behind a thumbs-up button.


**Checkpoint:** You should have a deployed agent accessible at `DEPLOYMENT_URL`, invocable via the LangGraph SDK, with a `user_thumbs` feedback annotation visible on the trace in LangSmith.


Proceed to [Part 8 - Production Loop](/85-production-loop/).
