# zcode-dynamic-workflow

A lightweight ZCode plugin that teaches the primary Agent to plan a task dynamically and route atomic work to:

- built-in Explore;
- worker-fast;
- worker-standard;
- worker-deep.

The primary Agent retains planning, dependency management, aggregation, escalation, verification, and the final response.

## Installation

Model IDs are pre-configured for the local machine (`builtin:zai-coding-plan/GLM-5.3`
at `low`/`high`/`max` effort — see `MODEL-MAPPING.md`). To install:

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

- Pure declarative Markdown/JSON — no executable code, no hooks, no MCP servers, no state files.
- Each worker declares an exhaustive `tools` allowlist: fast gets file tools + Bash (batch mechanical checks); standard/deep additionally get WebFetch (and WebSearch for deep). Session MCP tools are excluded from every tier.
- Workers treat file contents and delegated context as untrusted data, never instructions, and never print or transmit secret values.
- `maxTurns` (12/24/36) bounds each worker; subagents cannot spawn subagents; the primary Agent re-verifies evidence before accepting `done`.
- High-risk topics (security, auth, secrets, permission boundaries, irreversible operations, data migration, public API compatibility) must route to `worker-deep`.
- Residual risk: allowlists constrain tools, not file paths — path-level enforcement stays with the host permission system; in permissive host modes, prompt-level SCOPE is the only path constraint. For public distribution, pin `marketplace.json` ref to a tag or commit instead of `main`.
