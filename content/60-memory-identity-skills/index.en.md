---
title: "Memory, Identity, and Skills"
weight: 60
---

You'll route `/memories/` to S3 for per-customer durable memory, then add an always-loaded identity file (`AGENTS.md`) and on-demand skills that load only when relevant.

Why this matters: system prompts do not scale. `AGENTS.md` keeps the few always-on rules in one place. Skills give the agent task-specific playbooks without bloating every run. S3-backed memory lets customer facts survive across threads, kernel restarts, and redeploys.

## Long-Term Memory via S3

Route `/memories/` to S3 per customer. Everything outside `/memories/` stays ephemeral.

```python
import os
from tools import S3Store
from deepagents.backends import StateBackend, StoreBackend, CompositeBackend

mem_store = S3Store(bucket=os.environ["AGENT_FILES_BUCKET"], prefix="tour-memories")

def customer_memory_backend(customer_id: str) -> CompositeBackend:
    return CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(
                store=mem_store,
                namespace=lambda _rt, customer_id=customer_id: ("customers", customer_id, "memories"),
            )
        },
    )
```

Create an agent for one customer:

```python
customer_id = "C-1042"
other_customer_id = "C-2201"

mem_backend = customer_memory_backend(customer_id)

mem_agent = create_deep_agent(
    model=model,
    tools=[query_product_kb],
    system_prompt=(
        "Follow the user's file instructions exactly. Save durable facts to /memories/. "
        "Files outside /memories/ are ephemeral. When referencing file paths, use backticks."
    ),
    backend=mem_backend,
    store=mem_store,
    checkpointer=checkpointer,
)
```

Write a memory in one thread:

```python
memory_file = f"/memories/customer-{uuid7()}.md"
first_thread = {"configurable": {"thread_id": str(uuid7())}}

write_prompt = (
    f"Write `{memory_file}` with exactly: "
    "Customer C-1042 prefers email and has had 3 prior wifi tickets. "
    f"Then read `{memory_file}` back and quote its contents."
)

print("memory file:", memory_file)

write_state = mem_agent.invoke(
    {"messages": [{"role": "user", "content": write_prompt}]},
    config=first_thread,
)

print(write_state["messages"][-1].content)
```

Read the same memory from a fresh thread:

```python
second_thread = {"configurable": {"thread_id": str(uuid7())}}
read_prompt = f"Read `{memory_file}` from /memories/ and answer with only the saved customer fact."

read_state = mem_agent.invoke(
    {"messages": [{"role": "user", "content": read_prompt}]},
    config=second_thread,
)

print(read_state["messages"][-1].content)
```

Now try the same path with a different customer namespace:

```python
other_customer_agent = create_deep_agent(
    model=model,
    tools=[query_product_kb],
    system_prompt="Follow the user's file instructions exactly. When referencing file paths, use backticks.",
    backend=customer_memory_backend(other_customer_id),
    store=mem_store,
    checkpointer=checkpointer,
)

other_state = other_customer_agent.invoke(
    {"messages": [{"role": "user", "content": read_prompt}]},
    config={"configurable": {"thread_id": str(uuid7())}},
)

print(other_state["messages"][-1].content)
```

The original customer can read the memory from a fresh thread. The other customer cannot, because the namespace routes to a different S3 prefix.

## AGENTS.md: Always-Loaded Identity

`AGENTS.md` holds the agent's standing rules: role, workflow, house style, and constraints. Use it for the short list of instructions that should apply to every run.

Example rules from this workshop:

```md
## House style

- Lead with the answer, then the steps. No preamble.
- Plain language, second person, active voice.
- When referencing file paths, use backtick formatting like `path/file.md`, not markdown links.
```

In the notebook, `AGENTS.md` is loaded with:

```python
memory=["./AGENTS.md"]
```

## Skills: On-Demand Playbooks

A skill is a folder with a `SKILL.md`. The agent loads it only when its description matches the task. This progressive disclosure keeps the base prompt small while still giving the agent task-specific guidance.

The workshop's `support-reply` skill gives the final response structure: acknowledge, fix, close.

Seed the ticket and skill files into the agent's virtual filesystem:

```python
from deepagents.backends.utils import create_file_data

agents_md = (repo_root / "AGENTS.md").read_text()
skill_md = (repo_root / "skills" / "support-reply" / "SKILL.md").read_text()

ticket_md = '''# Ticket A-4471
Customer: C-1042
Order: A-4471
Product: SH-HUB-V2 SmartHome Hub
Issue: WiFi drops within 24 hours after setup.
Customer sentiment: frustrated after three prior wifi tickets.
Requested outcome: steps to try before replacement.
Preferred contact: email.
'''

seed_files = {
    "/AGENTS.md": create_file_data(agents_md),
    "/skills/support-reply/SKILL.md": create_file_data(skill_md),
    "/tickets/A-4471.md": create_file_data(ticket_md),
}
```

Build the skill-enabled agent:

```python
skill_agent = create_deep_agent(
    model=model,
    tools=[],
    system_prompt="You are a product support assistant. Read ticket files before drafting.",
    subagents=[researcher],
    memory=["./AGENTS.md"],
    skills=["./skills/"],
    backend=mem_backend,
    store=mem_store,
    checkpointer=checkpointer,
)
```

Ask it to read the ticket, delegate the KB lookup, and use the support-reply skill:

```python
result = skill_agent.invoke(
    {
        "messages": [{
            "role": "user",
            "content": (
                "Read `/tickets/A-4471.md`, delegate the SH-HUB-V2 wifi fix lookup to the "
                "researcher, then use the support-reply skill to draft the customer reply. "
                "Include the s3:// source URI."
            )
        }],
        "files": seed_files,
    },
    config={"configurable": {"thread_id": str(uuid7())}},
)

print(result["messages"][-1].content)
```

Use `AGENTS.md` for always-on identity and house rules. Use skills for task-specific playbooks. Use memory for durable customer facts across sessions.

Checkpoint: you should have a memory file persisted in S3 under one customer's namespace, readable from a fresh thread but isolated from another customer. The skill-enabled agent should produce a support reply following the support-reply structure without that full format living in the system prompt.

Proceed to [Part 6 - Evals](/70-evals/).
