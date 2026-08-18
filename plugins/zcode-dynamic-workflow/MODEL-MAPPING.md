# Model Mapping

Status: **configured and verified** (2026-08-19). The four Agent files use real model IDs
and were all exercised in the v0.1.3 regression session (docs/08 §11). Release `v0.1.5` is
published as a GPG-signed tag with the marketplace ref pinned to it; the
in-app GitHub update path was proven at v0.1.3.

## Active mapping (Option B — one model, three effort levels)

| Agent | model | thoughtLevel | maxTurns |
|---|---|---|---|
| worker-fast | `builtin:zai-coding-plan/GLM-5.3` | `low` | 12 |
| worker-standard | `builtin:zai-coding-plan/GLM-5.3` | `high` | 24 |
| worker-deep | `builtin:zai-coding-plan/GLM-5.3` | `max` | 36 |
| worker-review | `builtin:zai-coding-plan/GLM-5.3` | `max` | 30 |

The active provider on this machine is `builtin:zai-coding-plan` (Z.ai Coding Plan,
the only provider marked `available` in the local coding-plan cache). `GLM-5.3`
supports the reasoning levels `low`, `high`, and `max` according to the local model
registry, and the runtime logs show the qualified form
`builtin:zai-coding-plan/GLM-5.3` as the canonical model reference. Using one model
with three effort levels creates the fast/standard/deep tiers without depending on
multiple providers.

## Alternative mappings

### Option A — Multiple models

Other models visible on this machine but **not used** in the first version:

- `GLM-5.2` (1M context, no reasoning variants registered)
- `GLM-5-Turbo` / display name `glm-5-turbo` (200K context, reasoning variants
  `enabled`/`off`)
- `GLM-4.7` / display name `glm-4.7` (200K context)

A future refinement may move `worker-fast` to `glm-5-turbo` for genuinely lower
latency and cost, but its exact model-ID spelling and thoughtLevel values are not
yet confirmed by any local session log, so the first version keeps the single-model
mapping that is fully evidenced.

### Changing the mapping on another machine

1. Open ZCode model settings and copy the real model IDs (not display names).
2. Edit all `agents/worker-*.md` frontmatter: `model` and a supported `thoughtLevel`.
3. Remember: a subagent `thoughtLevel` only takes effect when `model` names a
   specific model; `model: inherit` follows the primary Agent and ignores it.
4. Start a new ZCode session so the changes load.
5. Invoke each worker with a trivial task and confirm the expected model and
   effort level are used.

## Validation

1. Search the plugin package for unresolved model-placeholder prefixes — the search must find nothing.
2. Start a new ZCode session.
3. Explicitly invoke each worker with a trivial task.
4. Confirm the expected model and effort are used.
