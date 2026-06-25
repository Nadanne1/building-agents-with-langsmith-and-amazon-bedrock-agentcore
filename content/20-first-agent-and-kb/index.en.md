---
title: "First Agent and Knowledge Base"
weight: 20
---

You'll build a Deep Agent that writes and reads from its virtual filesystem, queries a Bedrock Knowledge Base for product engineering issues, then delegates that lookup to a researcher sub-agent. This demonstrates the core pattern of planning, delegation, file handoffs, and context isolation.

Why this matters: a flat agent that calls every tool directly can bloat its own context with raw tool responses. Sub-agent delegation keeps the supervisor's context focused while the researcher does the KB lookup in its own window and returns a concise, cited summary.

## Verify the Environment

Confirm Bedrock model access is working:

```python
from langchain_aws import ChatBedrockConverse

model = ChatBedrockConverse(
    model="us.anthropic.claude-haiku-4-5-20251001-v1:0",
    region_name="us-east-1",
)

result = model.invoke("Say hi in exactly five words.")
print(result.content)
```

If this fails with `AccessDeniedException`, model access is not enabled for Claude Haiku 4.5 in `us-east-1`.

## Your First Deep Agent

Start with the harness before adding AWS tools. `create_deep_agent()` gives the agent planning, a virtual filesystem, and optional sub-agent delegation through one configured interface.

```python
from deepagents import create_deep_agent

agent = create_deep_agent(
    model=model,
    system_prompt=(
        "You are a helpful research assistant. "
        "When referencing file paths, use backtick formatting like `path/file.md`."
    ),
)

result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "Write a file called notes.md with 'Hello from Deep Agents on AWS!' then read it back to confirm."
    }]
})

print(result["messages"][-1].content)
```

Files live in the agent's virtual filesystem, not on disk yet:

```python
for path, fd in result.get("files", {}).items():
    raw_content = fd["content"] if isinstance(fd, dict) and "content" in fd else fd
    content = "\n".join(raw_content) if isinstance(raw_content, list) else str(raw_content)
    print(f"{path} -> {content}")
```

This is the core harness pattern you will reuse throughout the workshop: the agent can plan, write intermediate state to files, delegate focused work to sub-agents, and keep bulky context out of the supervisor conversation.

## Query the Knowledge Base

`query_product_kb` retrieves from a pre-provisioned Bedrock Knowledge Base seeded with product engineering docs for the SmartHome Hub, SmartCam, and SmartPlug.

```python
from tools import query_product_kb

evidence = query_product_kb.invoke("SH-HUB-V2 wifi drops firmware known issue fix")
print(evidence)
```

The tool returns up to 4 passages with `s3://` source citations, so the agent can cite the exact documented fix instead of guessing.

## Use the KB Tool Directly

Give the agent direct access to the KB tool first:

```python
agent = create_deep_agent(
    model=model,
    tools=[query_product_kb],
    system_prompt=(
        "You are a product support assistant. Use query_product_kb to look up documented "
        "product issues and fixes. Cite the fix exactly, and include a final 'Sources:' "
        "line with the s3:// source URI returned by the tool. When referencing file paths, "
        "use backticks."
    ),
)

result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "The SmartHome Hub (SKU SH-HUB-V2) keeps dropping wifi. Is there a known issue, what's the documented fix, and what source backs it?"
    }]
})

print(result["messages"][-1].content)
```

This direct-tool version is useful for simple questions. Next, you will move the lookup into a sub-agent so the supervisor can stay focused on planning and synthesis.

## Delegate to a Researcher Sub-Agent

The supervisor has no KB tool. It can only delegate product research to the researcher:

```python
researcher = {
    "name": "researcher",
    "description": "Looks up product engineering issues in the Bedrock Knowledge Base and returns cited findings.",
    "system_prompt": (
        "You are a product engineering researcher. Use query_product_kb for every product "
        "claim. Save exactly one note under /research/sh-hub-v2-wifi.md. Return only four "
        "fields: Issue, Fix, Source URI, and Saved file. Include the s3:// source URI."
    ),
    "tools": [query_product_kb],
}

agent = create_deep_agent(
    model=model,
    tools=[],
    subagents=[researcher],
    system_prompt=(
        "You are a product support supervisor. Use write_todos, delegate product lookups "
        "to the researcher sub-agent, read its /research/ notes, then answer the customer. "
        "When referencing file paths, use backticks."
    ),
)

result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "Research the SH-HUB-V2 wifi drop issue. Save one evidence note, then return the issue, documented fix, source URI, and saved file path."
    }]
})

print(result["messages"][-1].content)
```

Confirm the researcher saved its evidence note:

```python
print("Research files:")
for path in result.get("files", {}):
    if path.startswith("/research/"):
        print(" ", path)
```

## Inspect the Trace

Open `https://aws.smith.langchain.com` and find your trace in the `deepagents-aws-tour` project. Check three spans:

1. The `task` call - open the researcher subgraph to see its isolated context.
2. The `query_product_kb` tool call - confirm the returned passage includes an `s3://` source URI.
3. The final message - confirm it cites the documented fix and source.

Checkpoint: you should have an agent that writes to its virtual filesystem, queries the Bedrock Knowledge Base, delegates KB lookups to a researcher sub-agent, and produces a grounded answer citing the `s3://` source.

Proceed to [Part 2 - Pluggable Backends](/30-pluggable-backends/).
