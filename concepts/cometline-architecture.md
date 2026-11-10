# Cometline Three-Layer Architecture

The architectural pattern that separates Cometline into three layers with strictly one-way dependencies: desktop shell → agent runtime → LLM SDK.

## Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                         Desktop User                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  cometline (Electron + SvelteKit)                               │
│  - Chat UI, settings, animations                                │
│  - Sidecar lifecycle management                                 │
│  - Native OS integration (tray, shortcuts, auto-update)         │
└─────────────────────────────────────────────────────────────────┘
                              │ HTTP/SSE (127.0.0.1:7700)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  cometmind (Go runtime)                                         │
│  - Agent loop (up to 100 work steps + final answer)             │
│  - SQLite persistence (sessions, messages, memories)            │
│  - Built-in tools (file ops, commands, web fetch)               │
│  - Coding-harness CLI delegation (OpenCode, Claude Code, Codex) │
│  - MCP client manager and OAuth token refresh                    │
│  - Jobs, scheduled jobs, and autonomous job worker               │
│  - Discord gateway                                              │
│  - Semantic memory (retrieval, extraction, compaction)          │
└─────────────────────────────────────────────────────────────────┘
                              │ Provider interface
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  comet-sdk (Go library)                                         │
│  - Streaming LLM I/O (Anthropic, OpenAI, Codex, xAI, compatible)│
│  - Retry logic with exponential backoff                         │
│  - Tool-call assembly and delta tracking                        │
│  - Typed errors (auth, rate limit, server, stream)              │
└─────────────────────────────────────────────────────────────────┘
```

## Boundary Rules

### cometline (Desktop Shell)

**Owns:** Chat UI, settings management, session navigation, workspace switching, multimodal input, native desktop features, jobs board, skill draft editor, mini-window, file preview/editor.

**Must not own:** Tool execution, provider request construction, database writes, agent loop logic.

### cometmind (Agent Runtime)

**Owns:** Agent orchestration, session/workspace management, SQLite persistence, built-in tools, coding-harness CLI delegation, MCP server lifecycle, jobs/scheduling/autonomy, semantic memory, Discord gateway.

**Must not own:** Desktop windowing, renderer state, direct UI transitions.

### comet-sdk (LLM Library)

**Owns:** Provider-normalized requests, streaming events, retries, tool-call assembly, typed errors.

**Must not own:** Agent loops, tools, persistence, sessions.

## Key Contracts

- **HTTP/SSE API** at `127.0.0.1:7700` — contract between desktop and runtime
- **OpenAPI spec** at `cometmind/openapi.yaml` — source of truth, generates TS client + Go types
- **Provider interface** (`cometsdk.Provider`) — contract between runtime and SDK
- **SSE event stream** — real-time model/tool progress from runtime to UI

## Tech Stack

| Concern | Implementation | Swappable? |
|---|---|---|
| Provider-normalized LLM I/O | Go `comet-sdk` with multiple providers | Provider implementations behind `cometsdk.Provider` |
| Streaming collection | `comet-sdk/llm.StreamMessage` | If it preserves event ordering |
| Agent orchestration | `cometmind/internal/agent.Runner` | Load-bearing |
| Persistence | SQLite via `modernc.org/sqlite` + sqlc | Behind `session.Service` contracts |
| HTTP API | Gin in `cometmind/server` | If API/SSE contract preserved |
| Desktop shell | Electron main + SvelteKit renderer | Load-bearing |

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-project]]
