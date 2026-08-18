# Model Mapping

The three bundled Agent files contain placeholders and are not ready for final installation until they are replaced with real ZCode model IDs.

## Required replacements

```text
REPLACE_WITH_FAST_MODEL_ID
REPLACE_WITH_STANDARD_MODEL_ID
REPLACE_WITH_DEEP_MODEL_ID
```

## Option A — Multiple models

Choose:

- Fast: low-latency or low-cost model that can reliably do narrow tasks.
- Standard: balanced coding model used for most implementation.
- Deep: strongest available reasoning and coding model.

Example shape only:

```yaml
# worker-fast
model: <real-fast-model-id>
thoughtLevel: low

# worker-standard
model: <real-standard-model-id>
thoughtLevel: high

# worker-deep
model: <real-deep-model-id>
thoughtLevel: max
```

Do not assume the model's display name is its model ID.

## Option B — One model, three effort levels

Use the same concrete model ID for all three Agents and configure distinct supported thought levels.

For a model supporting `low`, `high`, and `max`:

```yaml
worker-fast: low
worker-standard: high
worker-deep: max
```

This is often the simplest MVP because it creates performance tiers without depending on several providers.

## Important ZCode behavior

- A subagent-specific `thoughtLevel` only takes effect when the Agent declares a specific model.
- With `model: inherit`, the subagent follows the primary Agent and ignores its own thought level.
- Supported levels depend on the model.
- Agent configuration changes should be tested in a new session.
- Plugin subagents may be read-only in the UI, so edit the Markdown files before packaging.

## Validation

After replacement:

1. Search the repository for `REPLACE_WITH_`.
2. Start a new ZCode session.
3. Explicitly invoke each worker with a trivial task.
4. Confirm the expected model and effort are used.
5. Record the mapping in the final implementation report.
