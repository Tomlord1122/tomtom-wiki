# Cometline Model Roles

Cometline's model configuration distinguishes one global **Default** model from optional model pins for helper roles. This prevents background work from accidentally inheriting a session's provider and makes fallback behavior explicit.

## Resolution rule

For every helper role, resolution is atomic:

1. If both a pinned `providerId` and `modelId` are present, use that exact pair.
2. If either value is missing, use `defaultProviderId` + `defaultModelId`.
3. Never combine a pinned provider with the Default model, or vice versa.

This rule is implemented by `Config.ResolveRoleLLM`. It is deliberately conservative: a half-configured role is not treated as a valid pin.

## Global Default

The shared settings file now uses:

```json
{
  "defaultProviderId": "openai",
  "defaultModelId": "gpt-4o"
}
```

`activeProviderId` is legacy read-only input. It may still be adapted for older installations, but new writes remove it. Runtime `Provider` and `Model` fields mirror the Default pair for compatibility with older callers.

The frontend model-role panel only offers explicitly enabled models. It also removes role pins that point to disabled or model-less providers, rather than leaving a stale configuration that fails later.

## Roles

The policy applies to:

- title generation;
- memory extraction and compaction;
- autonomy jobs;
- skill synthesis.

Memory extraction is especially important: it no longer borrows the provider/model from the active conversation. It uses the memory role pin when complete, otherwise Default. If no usable model exists, extraction is skipped with a diagnostic instead of attempting an ambiguous fallback. `NewMemoryLLM` and `memory_turn.go` both call `Config.ExtractionLLM()`, so an empty extraction pin still follows Default.

## Turn review follows the same fallback

Skill and wiki review now call `ExtractionLLM()`. An explicit memory pin wins; otherwise the Default model pair is used, matching memory extraction. A miss is logged as `skills.review.skipped_no_model` or `wiki.review.skipped_no_model` only after that review's own gate: ten user turns for wiki, ten tool-calling iterations for skills. `startTurnReview` checks the same resolver again and returns without a second log line.

An explicit pair such as `xai` / `grok-4.7` is used. Leaving the Memory extraction role empty now follows Default, which matches the settings-panel copy and the memory path. `TestSkillReviewUsesDefaultModelWhenExtractionUnpinned` locks that in. A runner with neither pin nor Default still skips.

A 2026-10-09 log and database check showed the same split in practice. Session `01M4E2XX2ZTVG79KH97QWSHQ0C` had `skill_review_mutating_count = 8`, so review had not started and no skip was logged. The only review warnings in `~/.cometmind/logs/cometline.log` were from 2026-10-08 on session `01M4CTS6TPW6ME2J5VP95YE8BT`, which had already crossed 10 turns and was skipped for a missing pin. That session is no longer in `cometmind.db`. The settings file at check time already had `extractionProviderId=xai` and `extractionModel=grok-4.7`.

## Why this exists

Session models are user-facing and temporary. Helper work is background infrastructure. Coupling the latter to the former creates surprising provider switches, fragile local/cloud boundaries, and inconsistent cost behavior. A single Default pair gives the runtime a stable baseline while still allowing explicit specialization.

The same boundary matters for Ollama: a role can remain pinned to a local provider, but an incomplete or invalid pin falls back as one unit rather than silently crossing providers.

## Related changes

The settings UI now explains title and memory roles as falling back to Default. See [[cometline-frontend]] for the settings pipeline and [[cometline-ollama-integration]] for local-provider behavior. The runtime path is part of [[cometline-agent-internals]] and [[cometline-semantic-memory]].

## References

- [[2026-07-21-cometline-update-default-roles-tooltips]]
- [[cometline-project]]
- [[cometline-frontend]]
- [[cometline-agent-internals]]
- [[cometline-semantic-memory]]
