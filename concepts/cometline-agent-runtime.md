# Cometline Agent Runtime

The Go-based agent execution engine (cometmind) that runs the multi-step agent loop, manages tools, and persists all state to SQLite.

## Agent Loop

The core orchestration in `cometmind/internal/agent.Runner`:

1. **Model call** — Send conversation + tool definitions + retrieved memories to the LLM provider via [[cometline-provider-sdk]]
2. **Persist turn** — Store user message, assistant response, and any tool calls in SQLite
3. **Execute tools** — If the model requests tool calls, execute them in order
4. **Continue** — Feed tool results back to the model, repeat (up to 100 work steps plus a final answer; the budget was raised from 50 to 100 in the August 2026 update)

All steps are streamed to the desktop shell as SSE events in real time.

## Built-in Tools

Tools implement `cometmind/internal/tools.Tool` with `Spec() ToolSpec` and `Execute(ctx, input)`. Registered in `registry.go`.

Core built-in tools:
- **File operations** — read, write, edit, list, glob, grep within workspace
- **Command execution** — shell commands with guardrails
- **Web fetch** — fetch public URLs
- **Web search** — DuckDuckGo search
- **Memory tools** — semantic memory CRUD (create, search, list, update, delete)
- **Skill tools** — load, create, edit, promote agent skills
- **Job tools** — propose, create, claim, update, complete jobs
- **MCP tools** — tools from connected MCP servers
- **Coding delegation** — `delegate_coding_task` to external harnesses

## Provider Support

The agent runtime maps provider IDs to comet-sdk providers:

| Cometline Provider | SDK Family | Notes |
|---|---|---|
| `anthropic` | Anthropic | Native Anthropic API |
| `openai` | OpenAI | Native OpenAI API |
| `openai-compatible` | OpenAI | Custom OpenAI-compatible endpoints |
| `opencode-go` | OpenAI | OpenCode Go runtime, dispatched by models.dev protocol metadata: `@ai-sdk/openai` → OpenAI Responses, `@ai-sdk/anthropic` → Anthropic Messages, default → Chat Completions |
| `openairesponses` | OpenAI Responses | API-key OpenAI Responses provider |
| `codex` | Codex | ChatGPT Codex subscription |
| `xai` | xAI | xAI Grok API |
| `ollama` | OpenAI | Loopback-only, via `/v1` compatibility layer |

Ollama chat uses the OpenAI-compatible API (native daemon base + `/v1`). Embeddings use native Ollama `/api/embeddings` directly. See [[cometline-ollama-integration]].

## Terminal Context

The agent runtime ingests terminal context attached to chat messages. Terminal selections use the `terminal://<sessionId>` source scheme. Server-side validation in `messages.go` requires this exact format. See [[cometline-terminal-panel]].

## Coding-Harness Delegation

The `delegate_coding_task` tool spawns an external coding agent (OpenCode, Claude Code, or Codex) through fixed, non-interactive CLI profiles managed by CometMind (`cometmind/internal/acp`). Users pick which harness; the command, arguments, and permissions are not user-editable.

Progress from the child process is streamed back to the parent session.

## MCP Client

CometMind connects to external Model Context Protocol servers and exposes their tools to the main agent. See [[cometline-mcp-integration]].

## Session & Workspace Model

- **Workspace** — a project directory with isolated chat history, tools, and memories
- **Session** — a conversation thread within a workspace, persisted to SQLite
- **Settings** — `~/.cometmind/cometline-settings.json` for runtime, `cometline-desktop.json` for UI

## Persistence

- **Database**: `~/.cometmind/cometmind.db` (SQLite, pure Go via `modernc.org/sqlite`)
- **Schema**: `cometmind/internal/db/schema.sql` with sqlc-generated Go code
- **Migrations**: `schemaVersion` + `alterStatements` in `internal/db/migrate.go` (current v26; verify before relying on the number)

## API

- **Protocol**: HTTP/SSE at `http://127.0.0.1:7700` (localhost only)
- **Framework**: Gin
- **Contract**: `cometmind/openapi.yaml` — generates TypeScript client + Go types

## Discord Gateway

The same agent runtime can run as a Discord bot with per-thread sessions and @mention gating. Started via `cometmind gateway run --platform discord`.

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-architecture]]
- [[cometline-project]]
