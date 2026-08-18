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
