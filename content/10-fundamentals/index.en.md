---
title: "Fundamentals"
weight: 10
---


This page introduces the core concepts you will use throughout the workshop.

## LangChain DeepAgents

DeepAgents is LangChain's multi-agent orchestration framework. Instead of a flat tool-calling loop, a DeepAgent uses:

- **A planner** (`write_todos`) that decomposes tasks into steps before acting
- **Sub-agents** with their own prompts and tool subsets, running in isolated context windows
- **A virtual filesystem** (`read_file`, `write_file`) as the handoff surface between sub-agents
- **Context management** - oversized tool results spill to the filesystem, conversation auto-summarizes near context limits

This pattern keeps the planner's context small while sub-agents do focused work.

## Amazon Bedrock

Fully managed access to foundation models:

- **Claude Haiku 4.5** - fast agent model (all parts)
- **Claude Sonnet 4.6** - eval judge (Part 6)
- **Titan Embeddings v2** - Knowledge Base embeddings

All accessed via cross-region inference profiles (the `us.` prefix in model IDs).

## Amazon Bedrock AgentCore

Managed runtime primitives for agents - tools you call on demand without provisioning:

- **Code Interpreter** - sandboxed Python execution in a MicroVM
- **Browser** - managed headless Chromium via Chrome DevTools Protocol
- **Gateway** - an MCP server that federates your Lambda APIs as discoverable tools with centralized auth

## Bedrock Knowledge Bases

Managed RAG: documents in S3 → chunking → Titan embeddings → OpenSearch Serverless. The agent calls `Retrieve` and gets passages with source citations.

## LangSmith (AWS-Region Instance)

Observability and evaluation at `aws.smith.langchain.com`:

- **Tracing** - hierarchical traces of every agent run
- **Datasets + Experiments** - curated examples and multi-evaluator scoring
- **Deployment** - managed runtime for LangGraph apps
- **Annotation queues** - human review feeding back into eval datasets

## MCP (Model Context Protocol)

A standard protocol for exposing tools to agents. AgentCore Gateway is an MCP server - you register Lambdas as targets and the agent discovers them over one connection, no per-tool wrapper code.

![Workshop Architecture](/static/architecture-diagram.png)

## Resources

- [LangChain DeepAgents Documentation](https://docs.langchain.com/deepagents)
- [Amazon Bedrock AgentCore](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [LangSmith Documentation](https://docs.smith.langchain.com/)

Proceed to [Part 1 - First Agent and KB](/20-first-agent-and-kb/).
