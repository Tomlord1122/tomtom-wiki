# Cometline Reasoning Replay Compatibility

DeepSeek family models on OpenCode's OpenAI-compatible endpoints require assistant tool-call history to preserve `reasoning_content`, including the empty string. This page records the wire-level compatibility rule and the reasoning continuation landscape.

## The DeepSeek contract

From the DeepSeek thinking-mode guide:

- Between two user messages without a tool call, prior `reasoning_content` does not need to be replayed.
- When an assistant message performed a tool call, its `reasoning_content` must participate in all subsequent interaction turns.
- An empty-string `reasoning_content` is meaningful and must be sent back verbatim.

A missing field triggers a 400 such as:

```text
The `reasoning_content` in the thinking mode must be passed back to the API.
```

This usually surfaces late in a long agent turn, because the full transcript is replayed on every step and the failing history was produced several steps earlier.

## Cometline's fix

The wire adapter now distinguishes absent from present-empty reasoning:

```go
// openAIMessage
Reasoning *json.RawMessage `json:"reasoning_content,omitempty"`
```

With preservation enabled, assistant messages that have no reasoning blocks still emit:

```json
"reasoning_content": ""
```

The policy is scoped by `modelcatalog.RequiresEmptyReasoningContentReplay`:

- provider method `opencode-go` / `opencode`
- model ID contains `deepseek`

`provider.NewForModel` applies `cometsdk.WithPreserveEmptyReasoningContent()` automatically, so [[cometline-agent-runtime]] stays provider-neutral.

## Continuation state across providers

Different reasoning providers need different replay contracts. This is why the fix is scoped instead of a global empty-string:

| Provider / protocol | Continuation requirement |
|---|---|
| Generic OpenAI Chat Completions | No reasoning replay needed |
| DeepSeek thinking (OpenAI-compatible) | Interleaved `reasoning_content`, preserve empty |
| Some compatible gateways | `reasoning_details`, `reasoning_text`, or vendor field |
| OpenAI Responses | Encrypted reasoning / provider state (`reasoning.encrypted_content`) |
| Anthropic / Bedrock extended thinking | Native reasoning block with signature / redacted data |
| Kimi / Moonshot | Provider-specific adaptive thinking semantics |

OpenCode models these as a model capability `interleaved: { field }` plus protocol-specific transforms; Cometline currently keeps the DeepSeek family rule hard-coded in the model catalog, which is enough until another provider demonstrates the same failure class.

## References

- [[2026-08-03-cometline-deepseek-reasoning-replay.md]]
- [[cometline-sdk-internals]]
- [[cometline-provider-sdk]]
- [[cometline-model-capability-learning]]
- [[cometline-agent-internals]]
- [[cometline-project]]
