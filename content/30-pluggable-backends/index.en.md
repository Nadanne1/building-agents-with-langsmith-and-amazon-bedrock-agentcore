---
title: "Pluggable Backends"
weight: 30
---


**You'll build:** Swap the agent's default ephemeral filesystem for durable storage - routing specific path prefixes to S3 via `CompositeBackend` while keeping scratch files ephemeral.

**Why this matters:** Production agents need files that survive across runs: post-mortems, customer context, shared artifacts. A backend decides where the virtual filesystem lives. Swap it without touching the agent logic.

---

## Backend Options

| Backend | Where Files Live | Lifetime |
|---------|------------------|----------|
| `StateBackend` (default) | LangGraph state | Ephemeral, per-thread |
| `FilesystemBackend` | Real disk / mounted volume (EFS) | Persistent on volume |
| `StoreBackend` | A LangGraph `BaseStore` (S3) | Durable, cross-thread |
| `CompositeBackend` | Routes path prefixes to the above | Mix and match |

---

## StateBackend: Ephemeral by Default

Files persist within a thread but vanish across threads:

```python
from langgraph.checkpoint.memory import MemorySaver
from deepagents import create_deep_agent

checkpointer = MemorySaver()
agent = create_deep_agent(model=model, tools=[query_product_kb],
    system_prompt="You are a product support assistant.",
    checkpointer=checkpointer)

```

---

## EFS Pattern: `FilesystemBackend`

`FilesystemBackend` writes real files under `root_dir`. On ECS/EKS/EC2 with Amazon EFS mounted, this gives persistent shared storage. Use `virtual_mode=True` to sandbox the agent.


EFS requires a mountable compute environment. In SageMaker Studio or CloudShell, treat this as the architecture pattern and use S3 for the hands-on persistence.


---

## S3 with CompositeBackend

Route `/durable/*` to S3, leave everything else ephemeral:

```python
from deepagents.backends import StateBackend, StoreBackend, CompositeBackend
from tools import S3Store

bucket = os.environ["AGENT_FILES_BUCKET"]
store = S3Store(bucket=bucket, prefix="deepagents-aws-tour")

backend = CompositeBackend(
    default=StateBackend(),
    routes={"/durable/": StoreBackend(store=store, namespace=lambda _rt: ("tour", "filesystem"))},
)

s3_agent = create_deep_agent(model=model, tools=[query_product_kb],
    system_prompt="Files under /durable/ persist to S3; everything else is ephemeral.",
    backend=backend, store=store, checkpointer=checkpointer)
```

After running, verify in S3:

```python
import boto3
s3 = boto3.client("s3", region_name="us-east-1")
resp = s3.list_objects_v2(Bucket=bucket, Prefix="deepagents-aws-tour")
for o in resp.get("Contents", []):
    print(o["Key"], f"({o['Size']} bytes)")
```

Only `/durable/` files appear in S3. Scratch files stayed in state and are gone next thread.


**Checkpoint:** You should see `/durable/findings.md` persisted in S3, while `/scratch.md` exists only in the LangGraph state and disappears on a new thread.


Proceed to [Part 3 - Browser and Code Interpreter](/40-browser-and-code-interpreter/).
