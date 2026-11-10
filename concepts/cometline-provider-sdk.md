# Cometline Provider SDK (comet-sdk)

The Go library (`comet-sdk`) that provides provider-agnostic LLM I/O — the foundation layer that isolates provider-specific details behind a stable interface.

## Provider Interface

All LLM providers implement `cometsdk.Provider`:

```go
type Provider interface {
    Stream(ctx context.Context, req Request) (<-chan StreamMessage, error)
    // ...
}
```

Supported providers:
- **Anthropic** — Claude models via API key
- **OpenAI** — GPT models via API key
- **OpenAI-compatible** — any OpenAI-compatible endpoint
- **OpenAI Responses** — API-key Responses provider sharing the `responsesproto` wire layer with Codex
- **OpenCode Go** — dispatched by models.dev protocol metadata (Responses / Anthropic / Chat Completions)
- **ChatGPT Codex** — subscription auth via `~/.codex/auth.json`
- **xAI Grok** — session token-based auth

## Key Capabilities

### Streaming

Returns a channel of `StreamMessage` events that the agent runtime consumes. Each event carries a delta (text, tool call partial, or reasoning block) plus metadata.

### Retry Logic

Exponential backoff with configurable retry policy. Handles transient failures (rate limits, server errors) transparently.

### Tool-Call Assembly

Accumulates streaming tool-call deltas into complete tool-call blocks that the agent runtime can execute. Tracks partial JSON and validates completion.

### Typed Errors

Classifies errors into:
- **Auth errors** — invalid/missing API key
- **Rate limit errors** — quota exceeded
- **Server errors** — provider-side 5xx
- **Stream errors** — connection drops mid-stream

### Reasoning continuation

Some thinking models require assistant tool-call history to keep their reasoning field on the wire, even when empty. `cometsdk.ProviderConfig.PreserveEmptyReasoningContent` (via `WithPreserveEmptyReasoningContent`) makes the OpenAI-compatible serializer emit `"reasoning_content": ""` instead of omitting the field. See [[cometline-reasoning-replay]].

## Testing

- **Unit/integration tests**: No live API calls (CI-safe)
- **Live tests**: Behind `live` build tag, require API keys
- **Fixtures**: Checked-in SSE fixtures under each provider's `fixtures/` directory for deterministic stream tests

## Adding a Provider

1. Implement `cometsdk.Provider` in `comet-sdk/provider/{name}/`
2. Add provider method constant in `cometmind/internal/config/config.go`
3. Wire into `cometmind/internal/provider/factory.go`
4. Add defaults/validation in `cometline/src/lib/settings/schema.ts`
5. Add UI behavior in `cometline/src/lib/components/settings/SettingsProvidersPanel.svelte`

## References

- [[2026-07-16-cometline-agents.md]]
- [[2026-08-03-cometline-deepseek-reasoning-replay.md]]
- [[cometline-architecture]]
- [[cometline-project]]
- [[cometline-reasoning-replay]]
