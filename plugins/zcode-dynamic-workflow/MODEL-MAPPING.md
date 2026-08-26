# Model Mapping

Status: **all five agents' mappings are configured and verified in live sessions: the
GLM-5.3 mapping on 2026-08-19 (docs/08 §11), and the GLM-5.3-Flash remap of
worker-fast/worker-standard plus the new `explore-flash` agent on 2026-08-27 in a
fresh ZCode 3.9.2 session: qualified model-ID resolution succeeded (all five agents
dispatched successfully in practice), and `low`/`high`/`max` showed no failures
attributable to effort values (actual effort-level effectiveness is not observable
in-session); evidence in docs/08-TEST-RECORD.md §21.** Release `v0.2.0` is published as
a GPG-signed tag with the marketplace ref pinned to it; the in-app GitHub update path
was proven at v0.1.3.

## Active mapping (hybrid — GLM-5.3-Flash for explore-flash/fast/standard, GLM-5.3 for deep/review)

| Agent | model | thoughtLevel | maxTurns |
|---|---|---|---|
| worker-fast | `builtin:zai-coding-plan/glm-5.3-flash` | `high` | 12 |
| worker-standard | `builtin:zai-coding-plan/glm-5.3-flash` | `max` | 24 |
| worker-deep | `builtin:zai-coding-plan/GLM-5.3` | `max` | 36 |
| worker-review | `builtin:zai-coding-plan/GLM-5.3` | `max` | 30 |
| explore-flash | `builtin:zai-coding-plan/glm-5.3-flash` | `low` | 16 |

The active provider on this machine is `builtin:zai-coding-plan` (Z.ai Coding Plan,
the only provider marked `available` in the local coding-plan cache). `GLM-5.3`
supports the reasoning levels `low`, `high`, and `max` according to the local model
registry, and the runtime logs show the qualified form
`builtin:zai-coding-plan/GLM-5.3` as the canonical model reference. The mapping now uses two models from that same provider: `GLM-5.3-Flash` for the
explore-flash/fast/standard tiers and `GLM-5.3` for the deep/review tiers.

2026-08-27: ZCode 3.9.2 provides GLM-5.3-Flash (model code `glm-5.3-flash`, from
ZCode model settings); the qualified form reuses the same `builtin:zai-coding-plan`
provider prefix, and the bare-ID fallback per the documented adjustment order is
`glm-5.3-flash`. The reasoning-effort field is `thoughtLevel` per the official
subagents documentation (https://zcode.z.ai/en/docs/subagents): no `reasoning_effort`
field exists, and unrecognized frontmatter keys are silently ignored, so only
`thoughtLevel` is written. Verified on 2026-08-27 in a fresh ZCode 3.9.2 session
(docs/08-TEST-RECORD.md §21): the qualified form resolved (all five agents were
dispatched successfully in practice), and `low`/`high`/`max` showed no failures
attributable to effort values (whether an effort level is actually in effect is not
observable in-session).

Also on 2026-08-27: added `explore-flash`, a fifth agent modeled on the built-in
Explore role for low-to-medium difficulty read-only exploration at lower cost
(`glm-5.3-flash`, `thoughtLevel: low` — adjusted from `high` the same day by user
decision, `maxTurns` 16, tools Read/Glob/Grep, no
shell/network/MCP). High-difficulty or broad exploration stays with built-in
`Explore`, with one-step escalation from `explore-flash`.

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
2. Edit all `agents/*.md` frontmatter: `model` and a supported `thoughtLevel`.
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
