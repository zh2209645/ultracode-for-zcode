---
description: Use a primary-agent-planned dynamic workflow and route atomic tasks to appropriately powered ZCode subagents.
argument-hint: "[task]"
skills: dynamic-workflow
---

Apply the mounted `dynamic-workflow` skill to the following task in orchestrator mode:

$ARGUMENTS

The primary Agent must retain ownership of decomposition, dependency waves, routing, aggregation, escalation, verification, and the final answer.

Orchestrator mode: first size the whole request against the skill's few-files threshold. Keep the primary context to the goal, task graph, decisions, results, and verified conclusions; leave file-heavy reading and intermediate detail to subagents — read-only discovery to built-in `Explore`, file changes to worker tiers, independent verdicts to `worker-review`. Do not execute a delegated-size task yourself unless it is trivial (delegation would cost more and be no more reliable), the subagent explicitly failed or returned incomplete work and re-dispatch or escalation cannot meet the acceptance criteria, or the work cannot be decomposed even after re-planning and couples several interrelated file modifications — execute that yourself to protect quality, and send significant or high-risk results through `worker-review`. Do not force delegation for trivial work, and do not assume every request needs parallel waves.
