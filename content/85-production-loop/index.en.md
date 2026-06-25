---
title: "Production Loop"
weight: 85
---


**You'll do:** Walk through the LangSmith UI surfaces that map to the agent lifecycle - traces, datasets, experiments, evaluators, annotation queues, and the deployment dashboard.

**Why this matters:** Build / Test / Deploy without Monitor is fire-and-forget. The production loop connects observation to improvement: observe traces → turn good and bad behavior into datasets → evaluate changes → route judgment-heavy cases for review → deploy when checks pass.

---

## The Surfaces

| Surface | What to Inspect |
|---------|-----------------|
| **Traces** | Tool calls, sub-agent dispatches, file writes, errors, latency, token use |
| **Datasets** | The `support-tickets-baseline` regression examples built from traces |
| **Experiments** | The `deepagents-aws-tour-smoke` run and evaluator feedback |
| **Evaluators** | LLM-as-judge safety checks + deterministic evidence and trajectory checks |
| **Annotation queues** | Manual human review for support replies that need judgment |
| **Deployment** | Studio, API docs, run logs, and the SDK invocation path |

---

## Walkthrough

1. Open your `deepagents-aws-tour` project URL and inspect the latest trace tree
2. Open **Datasets** and confirm `support-tickets-baseline` has the SH-HUB-V2 example
3. Open **Experiments** and find the `deepagents-aws-tour-smoke` run
4. Inspect the `required_evidence_present` evaluator feedback on the experiment row
5. Create or open the `support-ticket-review` annotation queue and add a trace when human review is needed

---

## The Production Loop

```
Observe traces
    → Turn good/bad behavior into dataset examples
        → Evaluate changes with the same evaluators
            → Route judgment-heavy cases to annotation queues
                → Deploy when checks pass
                    → Repeat
```

That's the agent development lifecycle in practice: the same agent harness from Part 1, now watched, measured, and continuously improved.


**Checkpoint:** You should be able to navigate the LangSmith UI and find your traces, dataset, experiment results, and the annotation queue. You understand how these surfaces connect into a continuous improvement loop.


Proceed to [Cleanup](/90-cleanup/).
