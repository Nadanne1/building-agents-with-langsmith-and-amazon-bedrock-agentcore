---
title: "Browser and Code Interpreter"
weight: 40
---


**You'll build:** Use AgentCore Browser to fetch and read a public support document, then use AgentCore Code Interpreter to run Python analytics in a managed sandbox - two managed tools that require zero infrastructure from you.

**Why this matters:** Browser gives the agent the ability to read real web content without you operating a headless-browser fleet. Code Interpreter lets it compute answers (counts, percentages, aggregations) in an isolated MicroVM instead of hallucinating math. Both are sessions you open on demand - nothing to deploy, nothing left running between calls.

---

## AgentCore Browser

The agent connects to managed Chromium over the Chrome DevTools Protocol via Playwright. A pre-provisioned support article in S3 provides realistic content without relying on external URLs.

```python
from tools import fetch_url, presign_public_support_doc

support_doc_url = presign_public_support_doc()
browser_text = fetch_url.invoke(support_doc_url)
print(browser_text[:1200])
```

In the LangSmith trace, `fetch_url` shows up as a tool call span. If this cell returns a retry message, the tool degrades gracefully - it returns a plain string instead of crashing the run.


No local Chromium install is required. Playwright connects to a remote browser via `connect_over_cdp()`, not `launch()`. The `playwright` Python package alone is enough.


---

## AgentCore Code Interpreter as a Sandbox Backend

A sandbox backend couples an isolated filesystem with an `execute` tool, so the agent writes and runs code in an AgentCore MicroVM - never on your machine. You pass it as `backend=`, not as a tool the model calls directly.

```python
from bedrock_agentcore.tools.code_interpreter_client import CodeInterpreter
from langchain_agentcore_codeinterpreter import AgentCoreSandbox

interp = CodeInterpreter(os.environ.get("AWS_REGION", "us-east-1"))
try:
    interp.start()
    code_agent = create_deep_agent(
        model=model,
        backend=AgentCoreSandbox(interpreter=interp),
        system_prompt="You can write and run Python in your sandbox to compute answers.",
    )
    result = code_agent.invoke({"messages": [{"role": "user",
        "content": "Of 47 support tickets this month, 18 were wifi-related. "
                   "Compute the percentage, rounded to one decimal."}]})
    print("answer:", result["messages"][-1].content)
finally:
    interp.stop()  # release the MicroVM
```

The sandbox reports **Linux** (AgentCore MicroVM on Amazon Linux); your machine reports its own OS. Proof the code executed remotely.


Always call `interp.stop()` in a `finally` block to release the MicroVM. Leaked sessions count against your account's concurrent-session quota.


---

## When to Use Which

- **CompositeBackend** (Part 2) - routes durable storage to S3/EFS. No code execution.
- **AgentCoreSandbox** (this part) - gives execution + an isolated filesystem. Pick it when the agent needs to compute answers.

Same agent, pick the backend that fits the job.


**Checkpoint:** You should have `fetch_url` returning visible text from the presigned support doc, and the Code Interpreter agent computing a correct percentage with proof it ran remotely (Linux kernel, not your local OS).


Proceed to [Part 4 - Gateway, MCP, and HITL](/50-gateway-mcp-and-hitl/).
