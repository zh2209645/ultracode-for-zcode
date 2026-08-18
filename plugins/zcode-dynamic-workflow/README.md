# zcode-dynamic-workflow

A lightweight ZCode plugin that teaches the primary Agent to plan a task dynamically and route atomic work to:

- built-in Explore;
- worker-review;
- worker-fast;
- worker-standard;
- worker-deep.

The primary Agent retains planning, dependency management, aggregation, escalation, verification, and the final response.

## Installation

Model IDs are pre-configured for the local machine (`builtin:zai-coding-plan/GLM-5.3`
at `low`/`high`/`max` effort — see `MODEL-MAPPING.md`). Release `v0.1.4` is published as
a GPG-signed tag and the marketplace source is pinned to it; v0.1.3 was verified in-app
from GitHub on ZCode 3.7.7 (mandatory regression 8/8 PASS, 2026-08-19). Protect release
tags from force-updates. To install:

1. In ZCode, open Settings → Plugins → Discover → `+`.
2. Add the marketplace repository: `https://github.com/zh2209645/ultracode-for-zcode`.
3. Install and enable `zcode-dynamic-workflow`.
4. Start a new session; the command, skill, three write workers, and read-only reviewer become available.

## Components

- `/ultracode [task]`
- `$dynamic-workflow`
- `worker-fast`
- `worker-standard`
- `worker-deep`
- `worker-review`

## Non-features

No hooks, MCP, runtime process, persistent state, Team server, Ralph loop, HUD, worktree manager, or nested delegation.

## Security posture

- Pure declarative Markdown/JSON — no bundled runtime or install-time executable, no hooks, no MCP servers, and no state files. Invoked workers can still modify files through their explicit file-tool allowlists.
- The three write workers declare `Read`, `Edit`, `Write`, `Glob`, and `Grep`; `worker-review` declares only `Read`, `Glob`, and `Grep`. All exclude `Bash`, `WebFetch`, `WebSearch`, and session MCP tools. Command execution, network research, and test/build verification stay with the primary Agent.
- Workers set `injectAgentsMd: false`. The primary Agent sends trusted control only through `TASK`, `SCOPE`, `CONSTRAINTS`, `ACCEPTANCE`, `VERIFY`, and `RETURN`; `CONTEXT`, repository quotations, upstream output, and file contents remain untrusted data, never instructions. Workers never print or copy secret values.
- `maxTurns` bounds all agents (12/24/36 for the three write workers and 30 for `worker-review`); subagents cannot spawn subagents; the primary Agent re-verifies evidence before accepting `done`.
- High-assurance independent review and adjudication use `worker-review`, which runs the configured high-performance model at maximum reasoning depth. General exploration, routine investigation, evidence collection, and ordinary analysis use Explore or stay with the primary Agent. Only high-risk or high-complexity tasks that explicitly require file changes route to `worker-deep`.
- At most 3 write-capable workers run concurrently. Genuinely independent read-only Explore or `worker-review` tasks may raise total concurrency to 10.
- Workspace `.zcode/**` is available only to the three write workers when the delegated task explicitly writes files, and only for task-owned todo/log records. Explore and `worker-review` report through the ZCode subagent result; `worker-review` cannot write files and must not read `.zcode`. User-level `~/.zcode`, plugin caches, credentials, unrelated session logs, and plugin-owned recovery databases remain out of scope. Shared `.zcode` files are updated serially; parallel write workers use distinct log files.
- Residual risk: allowlists constrain tools, not file paths. Use ZCode `Confirm Before Changes` (the documented default) or an equivalently restrictive workspace sandbox for worker writes; do not use write-capable workers in `Full Access` for untrusted repositories. Review the requested path before approving it.
- `marketplace.json` is pinned to the signed `v0.1.4` release tag. Protect or sign release tags, and bump the marketplace, plugin manifest, Skill metadata, and tag together for every release.
