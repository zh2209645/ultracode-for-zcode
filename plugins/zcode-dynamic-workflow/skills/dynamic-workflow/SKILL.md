---
name: dynamic-workflow
description: Use for substantive coding, refactoring, investigation, review, migration, or multi-part work where the primary Agent should decide whether to work directly or delegate atomic tasks to ZCode Explore, worker-fast, worker-standard, or worker-deep based on difficulty, risk, dependencies, and verification cost.
when_to_use: Use when work may benefit from isolated context, parallel subagents, model-tier routing, or dependency-aware execution. Do not use for a trivial single-step request that the primary Agent can complete faster and safely.
license: MIT
metadata:
  version: 0.1.0
---

# Dynamic Workflow

The primary Agent is the only orchestrator. Keep the user goal, task graph, dependencies, integration decisions, and final verification in the primary context.

## 1. Decide whether delegation helps

Work directly when the task is small, clear, low risk, and cheaper to complete than to delegate.

Use ZCode's built-in `Explore` for read-only repository search, call-chain mapping, implementation discovery, and evidence gathering.

Delegate only atomic tasks with clear scope and acceptance criteria.

## 2. Build a temporary task graph

For non-trivial work:

1. Identify the user goal and constraints.
2. Split the work into atomic tasks.
3. Mark dependencies and possible write conflicts.
4. Group independent tasks into execution waves.
5. Keep tasks that write the same file or depend on another result in separate waves.
6. Default to at most 10 concurrent subagents — launch only as many as genuine independence and value justify; the limit is a ceiling, not a target.

Dynamic planning belongs to the primary Agent. Do not use a fixed pipeline when the task does not need one.

## 3. Score each task

Score each dimension from 0 to 2:

- Scope: local/single-file, module-level, cross-module/system.
- Ambiguity: clear, some unknowns, unclear requirements or root cause.
- Reasoning/coupling: mechanical, normal reasoning, architectural or long-chain reasoning.
- Risk: non-behavioral, normal behavior, security/data/auth/API/production.
- Verification cost: one local check or a batch of mechanical per-item checks, several tests, broad regression or manual QA.

Default route:

- 0-2: primary Agent works directly.
- 3-4: `worker-fast`.
- 5-7: `worker-standard`.
- 8-10: `worker-deep`.

Overrides:

- Read-only codebase research (search, call-chain mapping, discovery, evidence gathering) -> built-in `Explore`. Rule-based verification that must run commands (per-entry checks, tests, builds) -> a worker tier, even when it writes nothing.
- Security, authentication, authorization, data migration, public API compatibility, architecture, or cross-module refactoring -> at least `worker-deep` analysis or independent deep review.
- Independent review of significant primary-Agent decisions, important changes, or acceptance results -> `worker-deep`.
- Mechanical edits, batch mechanical verification across several files or entries (for example per-entry manifest or checksum checks), small documentation changes, and precisely scoped low-risk work -> prefer `worker-fast`, but only when delegation is still justified under section 1 (never delegate a trivial task just to use this rule).
- Normal implementation, debugging, tests, and pattern-following multi-file changes -> prefer `worker-standard`.

Choose the lightest tier that can reliably satisfy the acceptance criteria.

## 4. Delegate with a complete task contract

Every delegated task must include:

- TASK: one clear action.
- SCOPE: allowed files/modules and explicit exclusions.
- CONTEXT: relevant findings, patterns, and upstream decisions.
- ACCEPTANCE: observable pass/fail conditions.
- VERIFY: checks the worker should run.
- RETURN: status, summary, files/evidence, verification, risks/blockers.

Do not delegate the entire user goal as one vague task.

## 5. Execute waves

Launch independent tasks together, up to the concurrency limit.

Use foreground parallel work when the next step needs all results. Use background work only when it is long-running and does not block the current path.

Subagents cannot spawn subagents. Workers must finish only their assigned task and return control to the primary Agent.

## 6. Integrate and re-plan

After each wave:

1. Check that results match the assigned scope.
2. Resolve contradictions and write conflicts in the primary context. If two workers return conflicting results, re-check the evidence; if still unresolved, spend that task's one escalation on a `worker-deep` independent adjudication.
3. Update the task graph when new information changes assumptions. Tasks whose upstream input ended `partial` or `blocked` must be re-scoped or held, not launched unchanged.
4. Do not trust a worker's `done` status without relevant evidence. A `done` returned without usable evidence: ask for the evidence once; if it is still absent, treat the task as `partial` and mark it unverified in the final answer. A worker that stops before returning STATUS also counts as `partial`.

Escalation:

- fast failure or expanded scope -> standard.
- standard failure, architectural issue, or high-risk discovery -> deep.
- deep failure -> primary Agent re-plans, narrows scope, or reports blocked.

Automatically escalate a task at most once, and do not repeatedly re-dispatch the same task to the same tier. Carry prior evidence and failure details into the upgraded prompt.

## 7. Verify by risk

Before the final answer:

- Small change: targeted diagnostics or relevant check.
- Standard implementation: affected tests plus typecheck/build when appropriate.
- High-risk or cross-module change: deep independent review and relevant tests.
- If a check cannot be run, state that it was not verified and why.

## 8. Worker result contract

Expected worker response:

```text
STATUS: done | partial | blocked

SUMMARY
- What was completed

FILES / EVIDENCE
- Changed files and key locations, or read-only evidence

VERIFICATION
- Commands/checks actually run and results
- Checks not run and reasons

RISKS / BLOCKERS
- Assumptions, residual risks, or missing prerequisites
```

Built-in `Explore` may return prose instead of this exact format, but its conclusions must still carry file, line, or command evidence.

## 9. Final response

The primary Agent should summarize:

- the dynamic plan used;
- which tasks were direct, Explore, fast, standard, or deep;
- files changed or evidence found;
- verification actually performed;
- unresolved risks or blocked items.

Before answering, collect any pending background results, or explicitly report them as still pending.
