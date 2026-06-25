---
title: "Pluggable Backends"
weight: 30
---

You'll build an agent that routes part of its virtual filesystem to durable S3 storage while keeping scratch files ephemeral.

Why this matters: production agents need some files to survive across runs, such as customer context, research notes, and shared artifacts. A backend decides where the virtual filesystem lives. You can swap or route backends without changing the agent's reasoning logic.

## Backend Options

| Backend | Where Files Live | Lifetime |
|---------|------------------|----------|
| `StateBackend` (default) | LangGraph state | Ephemeral, per-thread |
| `FilesystemBackend` | Real disk or mounted volume, such as EFS | Persistent on the volume |
| `StoreBackend` | A LangGraph `BaseStore`, such as S3 | Durable, cross-thread |
| `CompositeBackend` | Routes path prefixes to different backends | Mix and match |

S3 is the hands-on durable backend in this workshop. EFS is a pattern note unless your attendee environment can mount an EFS filesystem.

## StateBackend: Ephemeral by Default

The default backend stores files in LangGraph state. With a checkpointer, files persist within the same thread, but a new thread starts fresh.

```python
from langgraph.checkpoint.memory import MemorySaver
from langsmith import uuid7
from deepagents import create_deep_agent

checkpointer = MemorySaver()

agent = create_deep_agent(
    model=model,
    tools=[query_product_kb],
    system_prompt="You are a product support assistant. When referencing file paths, use backticks.",
    checkpointer=checkpointer,
)
```

Write a file and read it in the same thread:

```python
thread = {"configurable": {"thread_id": str(uuid7())}}

agent.invoke(
    {"messages": [{"role": "user", "content": "Write /research_notes.md with 'wifi issue confirmed for SH-HUB-V2'"}]},
    config=thread,
)

same_thread = agent.invoke(
    {"messages": [{"role": "user", "content": "Read /research_notes.md"}]},
    config=thread,
)

print("same thread:", same_thread["messages"][-1].content)
```

Now use a new thread:

```python
fresh = {"configurable": {"thread_id": str(uuid7())}}

new_thread = agent.invoke(
    {"messages": [{"role": "user", "content": "List files with ls /"}]},
    config=fresh,
)

print("new thread:", new_thread["messages"][-1].content)
```

The file exists in the original thread state, but the fresh thread has no memory of it.

## EFS Pattern: FilesystemBackend

`FilesystemBackend` writes real files under the `root_dir` you provide. On ECS, EKS, or EC2 with Amazon EFS mounted, this gives persistent shared storage. Use `virtual_mode=True` so file paths are scoped to that root.

EFS requires a compute environment that can mount NFS. In Workshop Studio, SageMaker Studio, or CloudShell-style environments without an EFS mount, treat EFS as the architecture pattern and use S3 for the hands-on durable backend.

## S3 with CompositeBackend

Use `CompositeBackend` to route `/durable/*` to S3 and leave everything else in ephemeral state.

```python
import os
from deepagents.backends import StateBackend, StoreBackend, CompositeBackend
from tools import S3Store

bucket = os.environ["AGENT_FILES_BUCKET"]
store = S3Store(bucket=bucket, prefix="deepagents-aws-tour")

backend = CompositeBackend(
    default=StateBackend(),
    routes={
        "/durable/": StoreBackend(
            store=store,
            namespace=lambda _rt: ("tour", "filesystem"),
        )
    },
)

s3_agent = create_deep_agent(
    model=model,
    tools=[query_product_kb],
    system_prompt=(
        "Files under /durable/ persist to S3; everything else is ephemeral. "
        "When referencing file paths, use backticks."
    ),
    backend=backend,
    store=store,
    checkpointer=checkpointer,
)
```

Write one durable file and one scratch file:

```python
s3_result = s3_agent.invoke(
    {
        "messages": [{
            "role": "user",
            "content": "Write /durable/findings.md with 'persisted to S3' and /scratch.md with 'ephemeral'."
        }]
    },
    config={"configurable": {"thread_id": str(uuid7())}},
)

print("agent wrote /durable/findings.md and /scratch.md")
```

Verify what reached S3:

```python
import boto3

s3 = boto3.client("s3", region_name=os.environ.get("AWS_REGION", "us-east-1"))
resp = s3.list_objects_v2(Bucket=bucket, Prefix="deepagents-aws-tour")

print("In S3:")
for o in resp.get("Contents", []):
    print(" ", o["Key"], f"({o['Size']} bytes)")
```

Inspect what stayed in LangGraph state:

```python
print("In LangGraph state:")
for path in s3_result.get("files", {}):
    print(" ", path)
```

Only `/durable/` files are routed to S3. Scratch files stay in state and are gone in a new thread.

Checkpoint: you should see `/durable/findings.md` persisted in S3, while `/scratch.md` exists only in LangGraph state for the current run.

Proceed to [Part 3 - Browser and Code Interpreter](/40-browser-and-code-interpreter/).
