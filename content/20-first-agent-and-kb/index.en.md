---
title: "First Agent and Knowledge Base"
weight: 20
---


**You'll build:** A DeepAgent that queries a Bedrock Knowledge Base for product engineering issues, then delegates that lookup to a researcher sub-agent - demonstrating the core pattern of planning, delegation, and context isolation.

**Why this matters:** A flat agent that calls tools directly bloats its own context with every raw response. Sub-agent delegation keeps the supervisor's context small - the researcher does the KB lookup in its own window and returns a concise, cited summary.

---

## Verify the Environment

Confirm Bedrock model access and LangSmith tracing are working:

```python
from langchain_aws import ChatBedrockConverse

model = ChatBedrockConverse(
    model="us.anthropic.claude-haiku-4-5-20251001-v1:0",
    region_name="us-east-1",
)
result = model.invoke("Say hi in five words.")
print(result.content)
```

If this fails with `AccessDeniedException`, model access is not enabled for Claude Haiku 4.5 in us-east-1.

---

## The KB Tool

`query_product_kb` retrieves from a pre-provisioned Bedrock Knowledge Base seeded with product engineering docs for the SmartHome Hub, SmartCam, and SmartPlug:

```python
from tools import query_product_kb

evidence = query_product_kb.invoke("SH-HUB-V2 wifi drops firmware known issue fix")
print(evidence)
```

The tool returns up to 4 passages with `s3://` source citations, so the agent cites the exact documented fix instead of guessing.

---

## Delegate to a Researcher Sub-Agent

The supervisor has no KB tool - it can only delegate to the `researcher`:

```python
from deepagents import create_deep_agent

researcher = {
    "name": "researcher",
    "description": "Looks up product engineering issues in the Bedrock Knowledge Base.",
    "system_prompt": (
        "You are a product engineering researcher. Use query_product_kb for every "
        "product claim. Save one note under /research/. Return Issue, Fix, Source URI."
    ),
    "tools": [query_product_kb],
}

agent = create_deep_agent(
    model=model,
    tools=[],
    subagents=[researcher],
    system_prompt=(
        "You are a product support supervisor. Use write_todos, delegate product "
        "lookups to the researcher, read its /research/ notes, then answer the customer."
    ),
)

result = agent.invoke({"messages": [{"role": "user",
    "content": "Research the SH-HUB-V2 wifi drop issue. Save one evidence note, "
               "then return the issue, documented fix, source URI, and saved file path."}]})
print(result["messages"][-1].content)
```

---

## Inspect the Trace

Open [aws.smith.langchain.com](https://aws.smith.langchain.com/) and find your trace in the `deepagents-aws-tour` project. Check three spans:

1. **The `task` call** - open the researcher subgraph to see its isolated context
2. **The `query_product_kb` tool call** - confirm the passage includes an `s3://` source URI
3. **The final message** - confirm it cites the documented fix and source


**Checkpoint:** You should have an agent that delegates KB lookups to a researcher sub-agent, with a trace in LangSmith showing the sub-agent's isolated tool calls and the grounded final answer citing the s3:// source.


Proceed to [Part 2 - Pluggable Backends](/30-pluggable-backends/).
