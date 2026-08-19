# ZCode Dynamic Workflow

`zcode-dynamic-workflow` is a lightweight, native ZCode plugin for dependency-aware task delegation. It lets the primary Agent keep control of the overall goal while routing each atomic subtask to the lightest capable execution path.

Current release: **v0.1.5** (published from the signed `v0.1.5` tag on 2026-08-19).

## What it does

The plugin adds a dynamic workflow policy that helps the primary Agent:

- size the whole request first and, once it exceeds reading or modifying a few files, plan a dynamic workflow before touching any file;
- keep its own context clean — goals, task graph, decisions, and verified conclusions instead of raw file bodies and exploration noise;
- complete trivial work directly instead of delegating it;
- use ZCode's built-in `Explore` agent for read-only discovery and evidence gathering;
- use `worker-fast` for simple, mechanical, low-risk changes;
- use `worker-standard` for normal implementation and test work;
- use `worker-deep` for clearly scoped, high-complexity file changes;
- use the read-only `worker-review` for high-assurance review, adjudication, and acceptance-evidence checks;
- run independent tasks in parallel and dependent tasks in ordered waves;
- escalate a failed write task at most once, then replan or report the blocker;
- verify evidence and produce the final response itself.

The primary Agent is always the sole orchestrator. Workers receive atomic task contracts and never take over the user's overall objective or create subagents of their own. Under `/ultracode` the primary Agent runs in orchestrator mode: execution is delegated by default, and it steps in itself only for trivial work, after a subagent explicitly fails or returns incomplete work that re-dispatch cannot recover, or when the work cannot be decomposed and couples several interrelated file changes.

## Routing model

| Work type | Default path |
|---|---|
| Trivial, local, low-risk change | Primary Agent |
| Read-only repository exploration | Built-in `Explore` |
| High-risk independent review or adjudication | `worker-review` |
| Simple, mechanical file change | `worker-fast` |
| Standard implementation or tests | `worker-standard` |
| Complex, cross-module, or high-risk file change | `worker-deep` |

Write tasks are scored by scope, ambiguity, coupling, risk, and verification cost. Task type overrides the score: discovery stays with `Explore`, ordinary analysis stays with the primary Agent, and high-assurance independent review goes to `worker-review`.

## Components

The release contains only declarative JSON and Markdown:

```text
marketplace.json
plugins/zcode-dynamic-workflow/
├── .zcode-plugin/plugin.json
├── commands/ultracode.md
├── skills/dynamic-workflow/SKILL.md
├── agents/
│   ├── worker-fast.md
│   ├── worker-standard.md
│   ├── worker-deep.md
│   └── worker-review.md
├── MODEL-MAPPING.md
├── README.md
└── LICENSE
```

It adds:

- the `/ultracode` command;
- the `dynamic-workflow` skill;
- three write-capable worker tiers;
- one read-only reviewer.

It does **not** add hooks, MCP servers, a Node/Python/TypeScript runtime, persistent workflow state, a Team or swarm service, a Ralph loop, a HUD, or automatic worktree management.

## Installation

1. Open **Settings → Plugins → Discover** in ZCode.
2. Select **+** and add this marketplace repository:

   ```text
   https://github.com/zh2209645/ultracode-for-zcode
   ```

3. Install and enable `zcode-dynamic-workflow`.
4. Start a new ZCode session so the command, skill, and agents are loaded.

The marketplace source is pinned to the signed `v0.1.5` release tag.

## Usage

Describe a multi-step task normally and allow the skill to trigger, or invoke it explicitly:

```text
/ultracode Refactor the authentication module, preserve API compatibility, and update the tests.
```

For small tasks, no special command is needed—the policy intentionally avoids unnecessary delegation. `/ultracode` activates orchestrator mode, where the primary Agent delegates execution by default, keeps its context to plans, decisions, and verified results, and completes work itself only when it is trivial, a subagent has explicitly failed or returned incomplete work that re-dispatch cannot recover, or the work is an unsplittable set of interrelated file changes.

## Model mapping

The published package uses one verified model with different reasoning levels:

| Agent | Model | Thought level | Max turns |
|---|---|---:|---:|
| `worker-fast` | `builtin:zai-coding-plan/GLM-5.3` | `low` | 12 |
| `worker-standard` | `builtin:zai-coding-plan/GLM-5.3` | `high` | 24 |
| `worker-deep` | `builtin:zai-coding-plan/GLM-5.3` | `max` | 36 |
| `worker-review` | `builtin:zai-coding-plan/GLM-5.3` | `max` | 30 |

Model availability is machine-specific. If your ZCode installation uses another provider, update the four agent frontmatter entries as described in [MODEL-MAPPING.md](plugins/zcode-dynamic-workflow/MODEL-MAPPING.md).

## Security boundaries

- All four agents set `injectAgentsMd: false`.
- Write workers allow only `Read`, `Edit`, `Write`, `Glob`, and `Grep`.
- `worker-review` allows only `Read`, `Glob`, and `Grep`.
- Workers have no shell, web, or MCP tools.
- Command execution, network access, and final test/build verification remain with the primary Agent.
- Write-capable workers should run under ZCode **Confirm Before Changes** or an equivalent restricted workspace mode, never unrestricted access in an untrusted repository.
- The plugin creates no recovery database or other persistent runtime state.

See [the delegation policy](docs/03-DELEGATION-POLICY.md) for the full task-contract, path-safety, concurrency, escalation, and evidence rules.

## Validation and release status

The plugin passed all eight mandatory behavioral scenarios on ZCode 3.7.7 using real plugin subagents: F-01, F-02, F-05, F-07, F-09, F-12, F-13, and F-14. Routing accuracy was 8/8, deep-worker misuse was 0%, and no nested delegation or infinite retry occurred.

The behavioral regression was executed on v0.1.3 (ZCode 3.7.7). Release v0.1.4 is a declarative documentation and release-integrity update with unchanged Skill, Command, and Agent behavior; it closed the final S-12 gate by synchronizing all version fields, pinning the marketplace to `v0.1.4`, and publishing a valid GPG-signed tag. Release v0.1.5 ships the request-sizing, context-hygiene, and orchestrator-mode prompt changes plus the English policy documents; the mandatory regression including the new F-15 scenario was re-run on v0.1.5 and passed in full — nine scenarios plus three fix-seam probes (2026-08-19, docs/08 §14). After the host updated to ZCode 3.8.1 (2026-08-19), core functionality — plugin loading, command/skill mounting, agent registration, and live Explore/worker-fast/worker-review dispatch — was re-verified in a fresh plugin session (docs/08 §15).

Evidence:

- [Acceptance criteria](docs/05-ACCEPTANCE-TESTS.md)
- [Test record](docs/08-TEST-RECORD.md)
- [Final implementation report](docs/09-FINAL-REPORT.md)
- [SHA-256 checksums](SHA256SUMS.txt)

## Documentation

- [MVP design and architecture](ZCODE_DYNAMIC_WORKFLOW_MVP.md)
- [Delegation policy](docs/03-DELEGATION-POLICY.md)
- [Migration and release plan](docs/04-MIGRATION-PLAN.md)
- [Model mapping](plugins/zcode-dynamic-workflow/MODEL-MAPPING.md)
- [Plugin package README](plugins/zcode-dynamic-workflow/README.md)

## License and attribution

This project is distributed under the MIT License. Its delegation concepts are inspired by [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode), also distributed under the MIT License. See [NOTICE.md](NOTICE.md) and the preserved [upstream license](references/omc/LICENSE).

This is an independent, unofficial ZCode plugin and is not affiliated with the ZCode or OMC teams.
