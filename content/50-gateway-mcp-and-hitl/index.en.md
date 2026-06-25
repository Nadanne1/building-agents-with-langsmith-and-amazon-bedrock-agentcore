---
title: "Gateway, MCP, and HITL"
weight: 50
---


**You'll build:** Discover order lookup, ticket history, and refund tools from AgentCore Gateway over MCP. Build a support triage agent with a Gateway-backed investigator and a Browser-backed researcher. Then gate the destructive `issue_refund` tool behind human approval.

**Why this matters:** Federating enterprise APIs through Gateway is the production-scaling story - once each backend is an MCP target, the agent code is identical no matter how many backends sit behind it. HITL is the production-safety story - agents that touch money need a halt-and-approve gate on irreversible actions.

---

## Discover Gateway Tools

The Gateway was registered during provisioning. Confirm the agent can discover the federated tools:

```python
from mcp_client import get_gateway_tools

gateway_tools = await get_gateway_tools()
print("Gateway tools:")
for t in sorted(gateway_tools, key=lambda tool: tool.name):
    print(f"  {t.name} - {(t.description or '')[:90]}")
```

You should see `lookup_order`, `lookup_customer_tickets`, and `issue_refund`.

---

## Build the Triage Agent

Combine Gateway-backed investigation with Browser-based research:

```python
from tools import fetch_url

by_name = {t.name: t for t in gateway_tools}

gateway_investigator = {
    "name": "gateway_investigator",
    "description": "Looks up orders and tickets through AgentCore Gateway, then checks the KB.",
    "system_prompt": "Look up the order, pull ticket history, query the KB for known issues...",
    "tools": [by_name["lookup_order"], by_name["lookup_customer_tickets"], query_product_kb],
}

browser_researcher = {
    "name": "browser_researcher",
    "description": "Reads public support docs through AgentCore Browser.",
    "system_prompt": "Use fetch_url to read public docs. Return only what the page says.",
    "tools": [fetch_url],
}

gateway_agent = create_deep_agent(
    model=model,
    tools=[*gateway_tools, query_product_kb, fetch_url],
    subagents=[gateway_investigator, browser_researcher],
    system_prompt="You are a smart-home support supervisor...",
    checkpointer=MemorySaver(),
    interrupt_on={"issue_refund": True},
)
```

---

## Run a Benign Ticket

```python
benign_state = await gateway_agent.ainvoke(
    {"messages": [{"role": "user", "content":
        "Order #A-4471 SmartHome Hub still drops wifi after setup. "
        "I tried the firmware update. What's the next step?"}]},
    config={"configurable": {"thread_id": str(uuid7())}},
)
print(benign_state["messages"][-1].content)
print("paused for approval:", bool(benign_state.get("__interrupt__")))
```

No interrupt - the ticket doesn't ask for a refund.

---

## Trigger Human-in-the-Loop on a Refund

Now run a ticket that explicitly requests a refund:

```python
from langgraph.types import Command

refund_state = await gateway_agent.ainvoke(
    {"messages": [{"role": "user", "content":
        "Order #A-4471 SmartHome Hub won't pair after firmware update. "
        "Third wifi issue across your devices. I want a full refund now."}]},
    config=refund_cfg,
)

if refund_state.get("__interrupt__"):
    action = refund_state["__interrupt__"][0].value["action_requests"][0]
    print("paused before tool:", action["name"])
    print("proposed args:", action["args"])

    # Approve the refund
    approved = await gateway_agent.ainvoke(
        Command(resume={"decisions": [{"type": "approve"}]}),
        config=refund_cfg,
    )
    print(approved["messages"][-1].content)
```

The Lambda is **never called** until a human approves. Try `"type": "reject"` or `"type": "edit"` with modified args to see different outcomes.


`interrupt_on={"issue_refund": True}` is one kwarg. The same `Command(resume=...)` shape works from a support UI, a Slack bot, or an approval workflow - not just stdin.


---

## What the Trace Shows

In LangSmith, the `issue_refund` span shows: original tool call → interrupt event → human decision → (if approved) Gateway/Lambda invocation. Every destructive operation has a recorded decision attached - that's the audit trail.


**Checkpoint:** You should have Gateway tools discovered via MCP, a benign ticket answered without interruption, and a refund ticket that pauses for human approval before the Lambda executes.


Proceed to [Part 5 - Memory, Identity, and Skills](/60-memory-identity-skills/).
