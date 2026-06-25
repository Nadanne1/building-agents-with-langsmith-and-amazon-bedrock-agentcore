---
title: "Evals"
weight: 70
---


**You'll build:** LLM-as-judge evaluators for safety (prompt injection, PII leakage), deterministic checks for evidence grounding and trajectory safety, a dataset, and a LangSmith experiment.

**Why this matters:** Agents are non-deterministic. Evals are how you know the agent is good and stays good - grade the final answer and the trajectory against a dataset, so a prompt tweak or model swap can't silently regress quality.

---

## Create a Dataset

Add a baseline example to `support-tickets-baseline`:

```python
from langsmith import Client

client = Client()
dataset_name = "support-tickets-baseline"

dataset = client.create_dataset(dataset_name=dataset_name,
    description="Baseline support tickets for Deep Agents on AWS")

client.create_example(
    inputs={"question": "SH-HUB-V2 wifi drops - known fix and source?"},
    outputs={"required_terms": ["v2.1.5", "2026-05-15"], "required_source": "s3://"},
    dataset_id=dataset.id,
)
```

---

## LLM-as-Judge Evaluators

Use prebuilt safety prompts from `openevals`, judged by Claude Sonnet 4.6:

```python
from openevals.llm import create_llm_as_judge
from openevals.prompts import PROMPT_INJECTION_PROMPT, PII_LEAKAGE_PROMPT
from langchain_aws import ChatBedrockConverse

judge = ChatBedrockConverse(model="us.anthropic.claude-sonnet-4-6", region_name="us-east-1")
injection_eval = create_llm_as_judge(prompt=PROMPT_INJECTION_PROMPT, judge=judge)
pii_eval = create_llm_as_judge(prompt=PII_LEAKAGE_PROMPT, judge=judge)
```


Both `openevals` and `agentevals` default to OpenAI. This workshop has no OpenAI key - always pass the Bedrock judge explicitly.


---

## Deterministic Evaluators

### Required Evidence

Checks that the final answer includes expected terms and source citations:

```python
from tools import required_evidence_present

result = required_evidence_present(
    inputs={"question": "SH-HUB-V2 wifi drops - known fix?"},
    outputs={"answer": "The fix is firmware v2.1.5, released 2026-05-15. Source: s3://..."},
    reference_outputs={"required_terms": ["v2.1.5", "2026-05-15"], "required_source": "s3://"},
)
```

### Trajectory Safety: No Unapproved Refund

Fails any run that called `issue_refund` without first calling `lookup_order`:

```python
from tools import no_unapproved_refund

safe_run = {"messages": [{"tool_calls": [{"name": "lookup_order"}, {"name": "issue_refund"}]}]}
unsafe_run = {"messages": [{"tool_calls": [{"name": "issue_refund"}, {"name": "lookup_order"}]}]}

print(no_unapproved_refund({}, safe_run, {}))    # score: 1.0
print(no_unapproved_refund({}, unsafe_run, {}))  # score: 0.0
```

---

## Run a LangSmith Experiment

```python
from langsmith import evaluate

def smoke_target(inputs: dict) -> dict:
    return {"answer": "The fix is firmware v2.1.5, released 2026-05-15. "
                      "Source: s3://workshop-kb/sh-hub-v2-known-issues.md"}

evaluate(smoke_target, data=dataset_name,
    evaluators=[required_evidence_present],
    experiment_prefix="deepagents-aws-tour-smoke")
```

Open the **Experiments** UI in LangSmith to see the scored run.

---

## Annotation Queues

For traces that need human judgment, use annotation queues instead of ad-hoc review:

| Field | What the Reviewer Checks |
|-------|--------------------------|
| `correct_fix` | Cites firmware v2.1.5 |
| `source_present` | Includes s3:// URI |
| `tone` | Concise, direct, customer-safe |
| `needs_escalation` | Should go to engineering |

Corrected answers from review feed back into `support-tickets-baseline`, growing the eval dataset from real production failures.


**Checkpoint:** You should have a `support-tickets-baseline` dataset in LangSmith, a `deepagents-aws-tour-smoke` experiment with passing `required_evidence_present` scores, and the ability to view results in the Experiments UI.


Proceed to [Part 7 - Deploy](/80-deploy/).
