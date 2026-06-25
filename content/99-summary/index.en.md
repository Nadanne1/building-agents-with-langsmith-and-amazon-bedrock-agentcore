---
title: "Summary"
weight: 99
---

You built the same agent harness, one capability at a time:

```python
agent = create_deep_agent(model=model)
agent = create_deep_agent(model=model, tools=[query_product_kb])
agent = create_deep_agent(model=model, subagents=[researcher])
agent = create_deep_agent(model=model, backend=CompositeBackend(...), store=mem_store)
agent = create_deep_agent(model=model, tools=[fetch_url])
agent = create_deep_agent(model=model, backend=AgentCoreSandbox(...))
agent = create_deep_agent(model=model, tools=[*gateway_tools, query_product_kb, fetch_url])
agent = create_deep_agent(model=model, interrupt_on={"issue_refund": True})
agent = create_deep_agent(model=model, skills=["./skills/"], memory=["./AGENTS.md"])
```

That's the workshop shape: configure the Deep Agents harness, then take it through Build -> Test -> Deploy Readiness -> Monitor.

## What You Covered

| Phase | Capabilities | AWS + LangChain |
|-------|--------------|-----------------|
| Build (Parts 1-5) | KB retrieval, sub-agent delegation, S3 backends, Browser, Code Interpreter, Gateway/MCP federation, HITL, memory, skills | Bedrock, AgentCore, S3, Lambda, Cognito, Deep Agents |
| Test (Part 6) | LLM-as-judge and deterministic evaluators, datasets, experiments | LangSmith + OpenEvals |
| Deploy Readiness (Part 7) | Local graph validation, `langgraph dev`, optional hosted deploy, optional SDK invocation and feedback | LangGraph dev + AWS LangSmith UI |
| Monitor (Part 8) | Traces, datasets, annotation queues, production loop | LangSmith |

## Key Takeaways

- Context isolation scales: sub-agents in their own context windows keep the planner coherent.
- Managed tools reduce undifferentiated operations: Code Interpreter, Browser, and Knowledge Bases are runtime capabilities, not browser or sandbox fleets you operate.
- MCP federation decouples agent code from backends: add a new Lambda target, and the agent discovers it through Gateway.
- HITL is one kwarg: `interrupt_on={"tool": True}` gives you an audited approval flow.
- Evals make quality measurable: LLM-as-judge and deterministic checks catch regressions.
- The production loop is continuous: observe, dataset, evaluate, review, validate or deploy, repeat.

## Resources

- [LangChain Deep Agents documentation](https://docs.langchain.com/deepagents)
- [Amazon Bedrock AgentCore documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [LangSmith documentation](https://docs.smith.langchain.com/)
- [Workshop source code](https://github.com/langchain-samples/langchain-aws-samples/tree/main/examples/deepagents-aws-tour)
