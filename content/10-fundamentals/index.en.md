---
title: "Fundamentals"
weight: 10
---

This page introduces the core concepts you will use throughout the workshop.

## LangChain Deep Agents

Deep Agents is LangChain's agent harness for long-running, multi-step work. Instead of a flat tool-calling loop, a Deep Agent can use:

- A planning tool (`write_todos`) to break work into steps before acting
- Sub-agents with their own prompts and tool subsets, running in isolated context windows
- A virtual filesystem (`read_file`, `write_file`, `edit_file`, `ls`, `grep`) as the handoff surface between agents
- Context management that moves oversized results into files and summarizes conversation history near context limits

This pattern keeps the supervisor's context focused while sub-agents do narrow research, tool use, and drafting work.

## Amazon Bedrock

Amazon Bedrock provides managed access to the foundation models used in this workshop:

- Claude Haiku 4.5 - fast agent model used throughout the notebook
- Claude Sonnet 4.6 - stronger judge model used for evals in Part 6
- Titan Embeddings v2 - embedding model used by the Bedrock Knowledge Base

The Claude model IDs use the `us.` cross-region inference profile prefix. The Knowledge Base uses Titan Embeddings v2 for document retrieval.

## Amazon Bedrock AgentCore

Amazon Bedrock AgentCore provides managed capabilities for agent workloads:

- Code Interpreter - sandboxed Python execution in an isolated MicroVM
- Browser - managed Chromium access through the Chrome DevTools Protocol
- Gateway - an MCP server that exposes Lambda-backed APIs as discoverable tools with centralized auth

In this workshop, Code Interpreter and Browser are used at runtime. Gateway is registered after infrastructure deploy with `scripts/register_gateway.py`.

## Bedrock Knowledge Bases

Bedrock Knowledge Bases provides managed retrieval over product support documents. The provisioning stack uploads documents to S3, indexes them with Titan Embeddings v2, and backs retrieval with OpenSearch Serverless.

The agent calls the Knowledge Base through `query_product_kb` and receives passages with source citations, so customer replies can cite the documented fix instead of guessing.

## LangSmith AWS-Region Instance

LangSmith provides observability, evaluation, and deployment workflows at `https://aws.smith.langchain.com`:

- Tracing - hierarchical traces for agent runs, sub-agent calls, tool calls, and file operations
- Datasets and experiments - curated regression examples and multi-evaluator scoring
- Evaluators - LLM-as-judge and deterministic checks for answer quality and trajectory safety
- Annotation queues - human review that can feed new examples back into datasets
- Deployment workflows - optional hosted deployment for LangGraph apps when workshop access allows it

## MCP

Model Context Protocol is a standard protocol for exposing tools to agents. AgentCore Gateway acts as an MCP server: you register Lambda targets, then the agent discovers order, ticket, and refund tools over one Gateway connection instead of writing one custom wrapper per backend API.

## Workshop Architecture

![Workshop Architecture](/static/langchain-aws-arch.png)

The workshop starts with a notebook running in the attendee environment. The notebook builds a Deep Agents support assistant that plans work, delegates research to sub-agents, uses a virtual filesystem for handoffs, and gates destructive refund actions with human approval.

The agent uses AWS services for the operational pieces:

- Bedrock Claude for model inference
- Bedrock Knowledge Bases for grounded product support retrieval
- S3 for durable agent files and customer-scoped memory
- AgentCore Browser for reading public support documentation
- AgentCore Code Interpreter for sandboxed Python execution
- AgentCore Gateway for Lambda-backed order, ticket, and refund tools over MCP
- Cognito for Gateway client-credentials auth

LangSmith receives traces from the notebook and is used for datasets, experiments, evaluators, annotation queues, and optional hosted deployment workflows.

## Resources

- [LangChain Deep Agents documentation](https://docs.langchain.com/deepagents)
- [Amazon Bedrock AgentCore documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [LangSmith documentation](https://docs.smith.langchain.com/)
- [Workshop source code](https://github.com/langchain-samples/langchain-aws-samples/tree/main/examples/deepagents-aws-tour)

Proceed to [Part 1 - First Agent and KB](/20-first-agent-and-kb/).
