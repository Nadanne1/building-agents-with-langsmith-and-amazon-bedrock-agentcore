---
title: "Memory, Identity, and Skills"
weight: 60
---


**You'll build:** Route `/memories/` to S3 for per-customer durable memory that survives across sessions. Add an always-loaded identity file (`AGENTS.md`) and on-demand skills that load only when relevant.

**Why this matters:** System prompts don't scale. Standing identity (`AGENTS.md`) keeps the few always-on rules tight. Skills give the agent task-specific playbooks without bloating every run. S3-backed memory means what the agent learns about a customer persists across restarts and redeploys.

---

## Long-Term Memory via S3

Route `/memories/` to S3 per customer. Everything outside stays ephemeral:

```python
from tools import S3Store
from deepagents.backends import StateBackend, StoreBackend, CompositeBackend

mem_store = S3Store(bucket=os.environ["AGENT_FILES_BUCKET"], prefix="tour-memories")

def customer_memory_backend(customer_id: str) -> CompositeBackend:
    return CompositeBackend(
        default=StateBackend(),
        routes={"/memories/": StoreBackend(
            store=mem_store,
            namespace=lambda _rt, cid=customer_id: ("customers", cid, "memories"),
        )},
    )
```

Write a memory in one thread, read it in a fresh thread - it persists because `/memories/` routes to S3:

```python
mem_agent = create_deep_agent(model=model, tools=[query_product_kb],
    system_prompt="Save durable facts to /memories/. Files outside are ephemeral.",
    backend=customer_memory_backend("C-1042"), store=mem_store, checkpointer=checkpointer)

```

A different customer ID gets a different S3 prefix - tenant isolation built into the namespace function.

---

## AGENTS.md: Always-Loaded Identity

`AGENTS.md` holds the agent's standing rules - who it is, house style, operating constraints. It's injected on every run via `memory=["./AGENTS.md"]`:

```markdown

## House style
- Lead with the answer, then the steps. No preamble.
- Plain language, second person, active voice.
- No corporate-speak.
```

---

## Skills: On-Demand Playbooks

A skill is a folder with a `SKILL.md` that the agent pulls in only when its description matches the task. Progressive disclosure keeps the system prompt small:

```python
skill_agent = create_deep_agent(model=model, tools=[],
    subagents=[researcher],
    memory=["./AGENTS.md"],
    skills=["./skills/"],
    backend=mem_backend, store=mem_store, checkpointer=checkpointer)
```

The `support-reply` skill provides reply structure (acknowledge → fix → close) without appearing in the system prompt. The agent loads it on demand because the task matches the skill's description.


**When to use which:** AGENTS.md for the short list of rules that apply to every run. Skills for task-specific playbooks. Memory for durable customer facts across sessions.



**Checkpoint:** You should have a memory file persisted in S3 under the customer's namespace, readable from a fresh thread. The skill-loaded agent should produce a reply following the support-reply structure without that format appearing in the system prompt.


Proceed to [Part 6 - Evals](/70-evals/).
