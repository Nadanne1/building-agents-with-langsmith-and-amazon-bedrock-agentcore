---
title: "Gateway, MCP, and HITL"
weight: 50
---

You'll discover order lookup, ticket history, and refund tools from AgentCore Gateway over MCP. Then you'll build a support triage agent with a Gateway-backed investigator and a Browser-backed researcher, and gate the destructive `issue_refund` tool behind human approval.

Why this matters: Gateway is the production-scaling pattern for enterprise APIs. Once each backend is exposed as an MCP target, the agent discovers tools through one connection instead of hardcoding a wrapper for every API. HITL is the production-safety pattern: agents that touch money need a halt-and-approve gate before irreversible actions.

## Discover Gateway Tools

The Gateway was registered during setup. Confirm the agent can discover the federated tools:

```python
from mcp_client import get_gateway_tools

gateway_tools = await get_gateway_tools()

print("Gateway tools:")
for t in sorted(gateway_tools, key=lambda tool: tool.name):
    print(" ", t.name, "-", (t.description or "")[:90])
```

You should see `lookup_order`, `lookup_customer_tickets`, and `issue_refund`.

## Build the Triage Agent

Combine Gateway-backed investigation with Browser-based public-doc research:

```python
from langgraph.checkpoint.memory import MemorySaver
from langsmith import uuid7
from tools import fetch_url

by_name = {t.name: t for t in gateway_tools}

gateway_investigator = {
    "name": "gateway_investigator",
    "description": (
        "Looks up orders and ticket history through AgentCore Gateway, then checks "
        "known product issues in the Bedrock Knowledge Base."
    ),
    "system_prompt": (
        "You are a support investigator. Look up the order, then use the customer_id "
        "from that order to pull ticket history. Query the product KB for known issues. "
        "Save one structured note under /research/gateway-investigation.md with Order, "
        "Customer History, Known Issues, and Sources sections."
    ),
    "tools": [by_name["lookup_order"], by_name["lookup_customer_tickets"], query_product_kb],
}

browser_researcher = {
    "name": "browser_researcher",
    "description": "Reads public support docs through AgentCore Browser when a ticket includes a URL.",
    "system_prompt": (
        "Use fetch_url to read public docs. Return only what the page actually says, "
        "include the URL, and say if it is not relevant."
    ),
    "tools": [fetch_url],
}

gateway_agent = create_deep_agent(
    model=model,
    tools=[*gateway_tools, query_product_kb, fetch_url],
    subagents=[gateway_investigator, browser_researcher],
    system_prompt=(
        "You are a smart-home support supervisor. Use write_todos. Delegate order, "
        "ticket, and KB fact gathering to gateway_investigator. Delegate public URLs "
        "to browser_researcher. Do not call lookup tools directly unless a demo asks "
        "you to. If the customer explicitly asks for a refund and the investigation "
        "supports it, call issue_refund with the order total and a concise reason. "
        "A human reviewer will approve, edit, or reject before the Gateway invokes "
        "the Lambda. Cite documented fixes exactly and use backticks for file paths."
    ),
    checkpointer=MemorySaver(),
    interrupt_on={"issue_refund": True},
)
```

## Run a Benign Ticket

This ticket asks for help, not a refund, so it should not pause for approval:

```python
benign_cfg = {"configurable": {"thread_id": str(uuid7())}}

benign_ticket = (
    "Order #A-4471 SmartHome Hub still drops wifi after setup. I tried the firmware "
    "update. I am frustrated and want to know the next step."
)

benign_state = await gateway_agent.ainvoke(
    {"messages": [{"role": "user", "content": benign_ticket}]},
    config=benign_cfg,
)

print(benign_state["messages"][-1].content)
print("paused for approval:", bool(benign_state.get("__interrupt__")))
```

Expected result: `paused for approval: False`.

## Trigger Human Approval on a Refund

For a live demo, use a focused refund agent so the interrupt path is deterministic. It still uses the real Gateway-backed `lookup_order` and `issue_refund` tools.

```python
from langgraph.types import Command

refund_only_agent = create_deep_agent(
    model=model,
    tools=[by_name["lookup_order"], by_name["issue_refund"]],
    system_prompt=(
        "Look up order A-4471 first. If the user explicitly asks for a refund, call "
        "issue_refund for the order total with a concise reason."
    ),
    checkpointer=MemorySaver(),
    interrupt_on={"issue_refund": True},
)

refund_cfg = {"configurable": {"thread_id": str(uuid7())}}

refund_state = await refund_only_agent.ainvoke(
    {"messages": [{"role": "user", "content": "Please process a full refund for order A-4471."}]},
    config=refund_cfg,
)

if refund_state.get("__interrupt__"):
    action = refund_state["__interrupt__"][0].value["action_requests"][0]
    print("paused before:", action["name"], action["args"])
else:
    print("did not interrupt:", refund_state["messages"][-1].content)
```

The Lambda has not run yet. Resume the same `thread_id` with a reviewer decision. The notebook rejects the refund for the live demo so attendees can see the pause and resume path without issuing a refund:

```python
rejected_state = await refund_only_agent.ainvoke(
    Command(resume={"decisions": [{"type": "reject", "message": "Reviewer rejected refund for demo."}]}),
    config=refund_cfg,
)

print(rejected_state["messages"][-1].content)
```

To test approval instead, rerun the trigger cell with a fresh `refund_cfg`, then resume that paused run with:

```python
Command(resume={"decisions": [{"type": "approve"}]})
```

Use the same `thread_id` when resuming. LangGraph needs it to find the paused run.

## What the Trace Shows

In LangSmith, the `issue_refund` path shows the proposed tool call, the interrupt, the human decision, and, if approved, the Gateway/Lambda invocation. That gives destructive operations an auditable decision trail.

Checkpoint: you should have Gateway tools discovered via MCP, a benign ticket answered without interruption, and a refund request that pauses for human approval before `issue_refund` can execute.

Proceed to [Part 5 - Memory, Identity, and Skills](/60-memory-identity-skills/).
