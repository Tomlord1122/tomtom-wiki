---
title: Cometline DeepSeek reasoning_content replay fix
source_url: file:///Users/tomlord/workspace/personal-project/cometline
source_revision: working tree after f7e1ee4 (uncommitted)
captured_at: 2026-08-03
---

# Captured source

Investigation and fix for a DeepSeek V4 thinking-mode 400 error during multi-step tool-call turns.

## Symptom

Long agent turns with `deepseek-v4-flash` through OpenCode Go failed late in the run:

```text
cometsdk: openai: server error (HTTP 400): Error from provider (Console Go):
Upstream request failed: [invalid_request_error]
The `reasoning_content` in the thinking mode must be passed back to the API.
```

The failure appeared at a later agent step (for example step 20) even though earlier tool steps succeeded, because the request replays the full session history and DeepSeek validates every prior assistant tool-call turn.

## Root cause

DeepSeek thinking mode requires that any assistant message which performed a tool call carry `reasoning_content` in every subsequent request. The API also treats an empty-string `reasoning_content` as semantically meaningful.

Cometline stored assistant reasoning in `messages.reasoning_content` as a JSON array, then serialized OpenAI-compatible history through `openAIMessage`:

```go
Reasoning any `json:"reasoning_content,omitempty"`
```

With no collected reasoning blocks, the field was omitted entirely because of `omitempty`. DeepSeek then rejected the tool-call continuation as missing required reasoning state. The agent loop, SQLite schema, and UI were all fine; the gap was at the wire adapter level.

## OpenCode context

The same bug class affects the broader ecosystem. OpenCode PR #24146 removed a truthy guard so `interleaved` fields are always written, preserving `reasoning_content: ""`. A follow-up commit (#24630) made the interleaved field default to `reasoning_content` for unknown OpenAI-compatible DeepSeek-family models (`apiID.includes("deepseek")`). OpenCode models this as a model capability (`interleaved: { field }`) rather than a DeepSeek-only special case, and keeps separate paths for Anthropic/Bedrock signatures and OpenAI Responses encrypted reasoning state.

## Fix implemented

- Added `cometsdk.ProviderConfig.PreserveEmptyReasoningContent` and `cometsdk.WithPreserveEmptyReasoningContent()`.
- Changed `openAIMessage.Reasoning` to a `*json.RawMessage` with `omitempty`, so the serializer can distinguish absent from present-empty. When preservation is enabled, assistant messages without reasoning blocks emit `"reasoning_content": ""`.
- `modelcatalog.RequiresEmptyReasoningContentReplay(method, providerID, modelID)` enables the policy only for OpenCode scoped providers (`opencode-go`, `opencode`) whose model ID contains `deepseek`.
- `provider.NewForModel` applies the option automatically when the policy matches, so the agent loop stays provider-neutral.

## Tests

- SDK: assistant tool-call message with empty reasoning serializes `reasoning_content: ""` when configured.
- Catalog: DeepSeek family (v4-flash, v4-pro, reasoner, r1) enables the policy; non-DeepSeek and unscoped custom gateways do not.

## Boundary

Other providers keep their current behavior. DeepSeek is the known contract requiring empty-field preservation; there is no generic continuation-policy framework yet, and one is not warranted until another provider demonstrates a similar requirement.
