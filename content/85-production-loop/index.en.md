---
title: "Production Loop"
weight: 85
---

You'll walk through the LangSmith UI surfaces that map to the agent lifecycle: traces, datasets, experiments, evaluators, annotation queues, and, if you completed the optional hosted path, the deployment dashboard.

Why this matters: Build, Test, and Deploy without Monitor is fire-and-forget. The production loop connects observation to improvement: observe traces, turn good and bad behavior into datasets, evaluate changes, route judgment-heavy cases for review, and validate or deploy when checks pass.

## The Surfaces

| Surface | What to Inspect |
|---------|-----------------|
| Traces | Tool calls, sub-agent dispatches, file writes, errors, latency, and token use |
| Datasets | The `support-tickets-baseline` regression examples built from traces |
| Experiments | The `deepagents-aws-tour-smoke` run and evaluator feedback |
| Evaluators | LLM-as-judge safety checks plus deterministic evidence and trajectory checks |
| Annotation queues | Manual human review for support replies that need judgment |
| Deployment | Optional hosted deployment: Studio, API docs, run logs, SDK invocation path, and feedback |

## Walkthrough

1. Open your `deepagents-aws-tour` project URL and inspect the latest trace tree.
2. Open **Datasets** and confirm `support-tickets-baseline` has the SH-HUB-V2 example.
3. Open **Experiments** and find the `deepagents-aws-tour-smoke` run.
4. Inspect the `required_evidence_present` evaluator feedback on the experiment row.
5. Create or open the `support-ticket-review` annotation queue and add a trace when human review is needed.
6. If you completed optional hosted deployment, open the deployment dashboard and check logs, API docs, SDK invocation details, and feedback.

## The Production Loop

```text
Observe traces
  -> Turn good and bad behavior into dataset examples
  -> Evaluate changes with the same evaluators
  -> Route judgment-heavy cases to annotation queues
  -> Validate or deploy when checks pass
  -> Repeat
```

That is the agent development lifecycle in practice: the same agent harness from Part 1, now watched, measured, and continuously improved.

Checkpoint: you should be able to navigate the LangSmith UI and find your traces, dataset, experiment results, and annotation queue. If you completed optional hosted deploy, you should also know where to find deployment logs and feedback.

Proceed to [Cleanup](/90-cleanup/).
