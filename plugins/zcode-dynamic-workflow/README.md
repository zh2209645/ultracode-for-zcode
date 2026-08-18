# zcode-dynamic-workflow

A lightweight ZCode plugin that teaches the primary Agent to plan a task dynamically and route atomic work to:

- built-in Explore;
- worker-fast;
- worker-standard;
- worker-deep.

The primary Agent retains planning, dependency management, aggregation, escalation, verification, and the final response.

## Before installation

Replace the model placeholders in `agents/*.md`. See `MODEL-MAPPING.md`.

## Components

- `/ultracode [task]`
- `$dynamic-workflow`
- `worker-fast`
- `worker-standard`
- `worker-deep`

## Non-features

No hooks, MCP, runtime process, persistent state, Team server, Ralph loop, HUD, worktree manager, or nested delegation.
