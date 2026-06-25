---
title: "Deep Agents on AWS"
weight: 0
---

Build a production-style customer support agent on AWS in a single notebook session. You'll walk through the full agent development lifecycle - Build -> Test -> Deploy -> Monitor - using LangChain Deep Agents for orchestration, LangSmith for observability, evals, and deployment workflows, and AWS services for inference, retrieval, managed tools, and compute.

The "better together" split: LangChain owns orchestration with Deep Agents, plus observability, evals, and deployment workflows through LangSmith. AWS owns inference with Bedrock Claude, retrieval with Bedrock Knowledge Bases, managed tools with AgentCore Browser, Code Interpreter, and Gateway, and backend compute with Lambda and S3.

## Workshop Structure

| Part | What You Build | AWS + LangChain |
|------|----------------|-----------------|
| 0 | Setup + verify | Bedrock + LangSmith tracing |
| 1 | First agent, KB tool, researcher delegation | Deep Agents + Bedrock KB |
| 2 | Pluggable backends: State, Filesystem, S3, Composite | S3 + EFS pattern |
| 3 | Managed Browser + Code Interpreter | AgentCore Browser + Code Interpreter |
| 4 | Federated order/ticket APIs + refund approval | AgentCore Gateway/MCP + HITL |
| 5 | Long-term memory, AGENTS.md, and skills | S3-backed Store |
| 6 | Evaluate with LLM-as-judge + deterministic checks | LangSmith + OpenEvals |
| 7 | Deploy readiness + optional hosted deploy | LangGraph dev + AWS LangSmith UI |
| 8 | Review production loop | LangSmith UI |

## Learning Objectives

By the end of this workshop, you will be able to:

- Build multi-agent systems with planning, delegation, and shared state using Deep Agents
- Query a Bedrock Knowledge Base from an agent sub-agent
- Route agent files to S3 for durable persistence across sessions
- Use AgentCore Browser to fetch and read public documentation
- Execute Python safely in AgentCore Code Interpreter sandboxes
- Federate Lambda APIs through AgentCore Gateway over MCP
- Gate destructive agent actions with human-in-the-loop approval
- Evaluate agent responses and trajectories with LangSmith
- Validate a LangGraph/LangSmith Deployment-ready agent locally and optionally deploy it through AWS LangSmith

## Duration

~90 minutes instructor-led, or self-paced.

Let's begin with the Prerequisites.
