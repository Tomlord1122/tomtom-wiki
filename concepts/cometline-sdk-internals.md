# Cometline SDK Internals (comet-sdk)

The foundational Go library (`github.com/cometline/comet-sdk`) that normalizes LLM provider communication behind a single interface.

## Module

```
github.com/cometline/comet-sdk
├── sdk.go              — Public API surface
├── errors.go           — Typed error hierarchy
├── llm/                — Shared types (StreamMessage, Request, etc.)
├── provider/           — Provider implementations
│   ├── anthropic/      — Claude models
│   ├── openai/         — GPT models (Chat Completions)
│   ├── openairesponses — OpenAI Responses API (API-key)
│   ├── codex/          — ChatGPT Codex (subscription auth, Responses wire)
│   └── xai/            — xAI Grok (session token auth)
└── internal/
    ├── providerbase/   — Shared provider helpers
    ├── responsesproto/ — Shared OpenAI Responses wire protocol (used by Codex + openairesponses)
    ├── retry/          — Exponential backoff
    └── sse/            — SSE parsing
```

## Core Interface

```go
type Provider interface {
    Stream(ctx context.Context, req Request) (<-chan StreamMessage, error)
}
```

All providers implement this. The agent runtime only sees `cometsdk.Provider` — it never knows which provider is active.

## Provider Implementations

### Anthropic (`provider/anthropic/`)

- API: Messages API with streaming
- Auth: `ANTHROPIC_API_KEY` or `COMETMIND_API_KEY`
- Features: reasoning blocks, tool use, extended thinking

### OpenAI (`provider/openai/`)

- API: Chat Completions with streaming
- Auth: `OPENAI_API_KEY` or `COMETMIND_API_KEY`
- Features: tool calls, reasoning, structured outputs

### OpenAI-compatible

- Any endpoint that speaks OpenAI's Chat Completions API
- Configurable `COMETMIND_BASE_URL`

### ChatGPT Codex (`provider/codex/`)

- Auth: Subscription-based OAuth session at `~/.codex/auth.json`
- No API key needed — uses local session token

### xAI Grok (`provider/xai/`)

- Auth: Borrowed Grok session token
- No official API key — uses session-based auth

## StreamMessage

The unified event type flowing from provider → agent runtime:

```go
type StreamMessage struct {
    Delta      string       // Text delta
    ToolCall   *ToolCall    // Tool call partial/complete
    Reasoning  *Reasoning   // Reasoning block delta
    FinishReason string     // stop, tool_calls, length, etc.
    Usage      *Usage       // Token usage when available
    Error      error        // Stream-level error
}
```

## Typed Error Hierarchy (`errors.go`)

| Error Type | HTTP Status | When |
|---|---|---|
| `AuthError` | 401/403 | Invalid/missing API key |
| `RateLimitError` | 429 | Quota exceeded |
| `ServerError` | 5xx | Provider-side failure |
| `StreamError` | N/A | Connection drop mid-stream |
| `BadRequestError` | 400 | Malformed request |

Each carries the original HTTP status, response body, and retryability flag.

## Retry Logic (`internal/retry/`)

Uses `github.com/cenkalti/backoff/v4`:
- Exponential backoff with jitter
- Configurable max retries and max elapsed time
- Retryable errors: `RateLimitError`, `ServerError` (5xx)
- Non-retryable: `AuthError`, `BadRequestError`
- Stream errors: retry with fresh connection

## SSE Parsing (`internal/sse/`)

Shared SSE parser used by all providers that stream via Server-Sent Events. Handles:
- Chunked transfer encoding
- Multi-line events
- Incomplete line buffering
- Provider-specific event formats normalized to `StreamMessage`

## Testing

- **Unit tests**: `*_test.go` files in each package, CI-safe (no live API)
- **Fixtures**: Checked-in SSE fixture files in `provider/*/fixtures/` for deterministic stream tests
- **Live tests**: Behind `live` build tag (`go test -tags=live`), require API keys
- **Make targets**: `make test`, `make test-live`, `make test-live-anthropic`, `make test-live-openai`

## Dependencies

Minimal by design — only `cenkalti/backoff` for retries and `stretchr/testify` for testing. No external HTTP client (uses `net/http`), no JSON library (uses `encoding/json`).

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-architecture]]
- [[cometline-provider-sdk]]
- [[cometline-project]]
