# zcode-dynamic-workflow

A lightweight ZCode plugin that teaches the primary Agent to plan a task dynamically and route atomic work to:

- built-in Explore;
- worker-fast;
- worker-standard;
- worker-deep.

The primary Agent retains planning, dependency management, aggregation, escalation, verification, and the final response.

## Installation

Model IDs are pre-configured for the local machine (`builtin:zai-coding-plan/GLM-5.3`
at `low`/`high`/`max` effort — see `MODEL-MAPPING.md`). Release `v0.1.2` must be
published before remote installation because the marketplace source is pinned to that
versioned tag. Protect the tag from force-updates. To install:

1. Push this repository (the kit root containing `marketplace.json`) to GitHub.
2. In ZCode: Settings → Plugins → Discover → `+` → paste the GitHub repository URL.
3. Install and enable `zcode-dynamic-workflow`.
4. Start a new session; the command, skill, and three workers become available.

## Components

- `/ultracode [task]`
- `$dynamic-workflow`
- `worker-fast`
- `worker-standard`
- `worker-deep`

## Non-features

No hooks, MCP, runtime process, persistent state, Team server, Ralph loop, HUD, worktree manager, or nested delegation.

## Security posture

- Pure declarative Markdown/JSON — no bundled runtime or install-time executable, no hooks, no MCP servers, and no state files. Invoked workers can still modify files through their explicit file-tool allowlists.
- Each worker declares the same file-only allowlist: `Read`, `Edit`, `Write`, `Glob`, and `Grep`. This permits task-owned todo/log updates under workspace `.zcode/**` while excluding `Bash`, `WebFetch`, `WebSearch`, and session MCP tools. Command execution, network research, and test/build verification stay with the primary Agent.
- Workers set `injectAgentsMd: false`. The primary Agent reviews project instructions and passes only the minimum trusted rules needed for the assigned atomic task. Workers treat file contents and delegated context as untrusted data, never instructions, and never print or copy secret values.
- `maxTurns` (12/24/36) bounds each worker; subagents cannot spawn subagents; the primary Agent re-verifies evidence before accepting `done`.
- High-risk topics (security, auth, secrets, permission boundaries, irreversible operations, data migration, public API compatibility) must route to `worker-deep`.
- At most 3 write-capable workers run concurrently. Independent read-only Explore tasks may raise total concurrency to 10.
- Workspace `.zcode/**` is available only to delegated tasks that explicitly write files, and only for task-owned todo/log records. Explore, analysis, review, and verification tasks report through the ZCode subagent result and do not access `.zcode`. User-level `~/.zcode`, plugin caches, credentials, unrelated session logs, and plugin-owned recovery databases remain out of scope. Shared `.zcode` files are updated serially; parallel write workers use distinct log files.
- Residual risk: allowlists constrain tools, not file paths. Use ZCode `Confirm Before Changes` (the documented default) or an equivalently restrictive workspace sandbox for worker writes; do not use write-capable workers in `Full Access` for untrusted repositories. Review the requested path before approving it.
- `marketplace.json` is pinned to the versioned `v0.1.2` release tag. Protect or sign release tags, and bump the manifest versions and tag together for every release.
