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

That's the workshop shape: configure the DeepAgents harness, then take it through Build → Test → Deploy → Monitor.

## What You Covered

| Phase | Capabilities | AWS Services |
|-------|-------------|--------------|
| **Build** (Parts 1-5) | KB retrieval, sub-agent delegation, S3 backends, Browser, Code Interpreter, Gateway/MCP federation, HITL, memory, skills | Bedrock, AgentCore, S3, Lambda, Cognito |
| **Test** (Part 6) | LLM-as-judge + deterministic evaluators, experiments | LangSmith |
| **Deploy** (Part 7) | One-command deploy, SDK invocation, feedback | LangSmith Deployment |
| **Monitor** (Part 8) | Traces, datasets, annotation queues, production loop | LangSmith |

## Key Takeaways

1. **Context isolation scales** - sub-agents in their own context windows keep the planner coherent
2. **Managed tools eliminate undifferentiated ops** - Code Interpreter, Browser, and KB are sessions, not infrastructure
3. **MCP federation decouples agent code from backends** - add a new Lambda target, the agent discovers it
4. **HITL is one kwarg** - `interrupt_on={"tool": True}` gives you an audited approval flow
5. **Evals make quality measurable** - LLM-as-judge + deterministic checks catch regressions
6. **The production loop is continuous** - observe, dataset, evaluate, review, deploy, repeat

## Resources

- [LangChain DeepAgents Documentation](https://docs.langchain.com/deepagents)
- [Amazon Bedrock AgentCore](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [LangSmith Documentation](https://docs.smith.langchain.com/)
- [Workshop Source Code](https://github.com/srimanthtangedipalli-eng/deepagents-aws-tour)
