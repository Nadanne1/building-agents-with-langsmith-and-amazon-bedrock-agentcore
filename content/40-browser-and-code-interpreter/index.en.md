---
title: "Browser and Code Interpreter"
weight: 40
---

You'll use AgentCore Browser to fetch and read a public support document, then use AgentCore Code Interpreter to run Python analytics in a managed sandbox.

Why this matters: Browser lets the agent read web content without you operating a headless-browser fleet. Code Interpreter lets the agent compute answers in an isolated MicroVM instead of guessing at counts, percentages, or aggregations. Both are opened on demand at runtime.

## AgentCore Browser

AgentCore Browser gives the agent managed Chromium without running a browser on your machine. The workshop uses a pre-provisioned support article in S3 and generates a temporary URL, so the Browser reads realistic support content without relying on an external website for a fictional company.

```python
from tools import fetch_url, presign_public_support_doc

support_doc_url = presign_public_support_doc()
browser_text = fetch_url.invoke(support_doc_url)
print(browser_text[:1200])
```

In the LangSmith trace, `fetch_url` appears as an AgentCore Browser tool call span. If this cell returns a retry message, the rest of the agent can still proceed: the tool degrades to a plain string instead of crashing the run.

No local Chromium launch is required. The tool connects to managed Chromium over the Chrome DevTools Protocol with Playwright's `connect_over_cdp()`.

## AgentCore Code Interpreter as a Sandbox Backend

A sandbox backend couples an isolated filesystem with an `execute` tool. The agent writes and runs code in an AgentCore MicroVM, not on your host. You pass the sandbox as `backend=`, rather than exposing Code Interpreter as a normal tool.

Always call `stop()` in a `finally` block to release the MicroVM.

```python
import os
import platform
from bedrock_agentcore.tools.code_interpreter_client import CodeInterpreter
from langchain_agentcore_codeinterpreter import AgentCoreSandbox

interp = CodeInterpreter(os.environ.get("AWS_REGION", "us-east-1"))

try:
    interp.start()
    code_agent = create_deep_agent(
        model=model,
        backend=AgentCoreSandbox(interpreter=interp),
        system_prompt=(
            "You can write and run Python in your sandbox to compute answers. "
            "When referencing file paths, use backticks."
        ),
    )

    result = code_agent.invoke({
        "messages": [{
            "role": "user",
            "content": "Of 47 support tickets this month, 18 were wifi-related. Write and run Python to compute the percentage, rounded to one decimal."
        }]
    })

    print("answer:", result["messages"][-1].content)

    proof = code_agent.invoke({
        "messages": [{
            "role": "user",
            "content": "Run Python that prints platform.system(), platform.release(), and socket.gethostname()."
        }]
    })

    print("sandbox reports:", proof["messages"][-1].content)
    print("this machine:   ", platform.system(), "/", platform.node())
finally:
    try:
        interp.stop()
    except Exception as stop_err:
        print(f"cleanup warning: {stop_err}")
```

The sandbox should report Linux from the AgentCore MicroVM. Your local machine reports its own OS and hostname. That contrast proves the code executed remotely.

## When to Use Which Backend

- `CompositeBackend` from Part 2 routes files to durable storage such as S3 or EFS. It does not provide code execution.
- `AgentCoreSandbox` provides code execution plus an isolated filesystem. Use it when the agent needs to compute answers safely.

Same agent harness, different backend choice. Pick the backend that fits the job.

Checkpoint: `fetch_url` should return visible text from the presigned support doc, and the Code Interpreter agent should compute the correct percentage with proof that the code ran in a remote Linux MicroVM.

Proceed to [Part 4 - Gateway, MCP, and HITL](/50-gateway-mcp-and-hitl/).
