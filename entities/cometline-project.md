# Cometline

A local-first AI companion desktop application with a three-layer architecture: a SvelteKit + Electron desktop shell, a Go agent runtime, and a provider-agnostic LLM SDK.

## Summary

Cometline lets users run a local desktop AI assistant with persistent workspace-scoped sessions, visible streaming reasoning/tool activity, provider switching, and a native-feeling desktop UI. The agent runtime stays outside the renderer for security.

## Key Properties

- **Local-first**: Everything runs on the user's machine. SQLite persistence, no cloud dependency.
- **Workspace-scoped**: Chat history, sessions, tools, and memories are isolated per project directory.
- **Multi-persona**: Switch between companion personalities (Minako, Souma) with distinct avatars, tone, and system prompts.
- **Multi-provider**: Supports Anthropic, OpenAI, OpenAI-compatible APIs, ChatGPT Codex (subscription), xAI Grok, and Ollama for local/private use.
- **Self-improving**: Can improve its own codebase using coding-harness delegation.

## Three Modules

| Module | Language | Role |
|---|---|---|
| [[cometline-sdk-internals]] | Go library | Provider-agnostic LLM I/O |
| [[cometline-agent-runtime]] | Go binary | Agent loop, persistence, API, tools |
| [[cometline-desktop-shell]] | SvelteKit + Electron | Chat UI, settings, native shell |

Dependency direction: `cometline` → `cometmind` → `comet-sdk`

## Distribution

- macOS 13+ via Homebrew: `brew install --cask cometline/tap/cometline`
- Signed, notarized, with auto-update via GitHub Releases
- Apple Silicon and Intel supported

## Architecture

See [[cometline-architecture]] for the full three-layer design.

## Key Features

- [[cometline-agent-runtime]] — Multi-step agent loop with built-in tools
- [[cometline-semantic-memory]] — Auto-retrieve/extract/compact across sessions
- [[cometline-mcp-integration]] — External MCP server connections with OAuth
- [[cometline-jobs-scheduling]] — Durable work queue, cron scheduling, autonomy
- [[cometline-provider-sdk]] — Streaming LLM I/O with retries and typed errors
- [[cometline-desktop-shell]] — Native shell with mini-window, jobs board, routing
- [[cometline-web-panel-and-wiki]] — Workspace file tree, previews, wikilinks, backlinks, and @mentions
- [[cometline-inbox-system]] — Asynchronous notifications, replies, and background processing
- [[cometline-model-capability-learning]] — Adaptive provider/model capability compatibility
- [[cometline-turn-reliability]] — Stream recovery, incomplete tool calls, and user nudges
- [[cometline-ollama-integration]] — Local LLM support via Ollama daemon with cataloged models
- [[cometline-terminal-panel]] — In-app terminal via xterm.js + Electron PTY, scoped per session
- [[cometline-model-roles]] — Default model pair and atomic helper-role resolution
- [[cometline-agent-controls]] — Agent mode boundaries and provider-validated reasoning effort

## Current snapshot

See [[cometline-current-platform-state]]. As of merge `3206abcb` (2026-10-08, PR 194 merged to `main`), the August controls, protocol routing, live workspace tree, and session lifecycle are still in place. Wiki compile and skill review share one hidden child, one at a time. Wiki review waits for ten user turns; skill review waits for ten tool-calling iterations. `query-wiki` stays read-only, and the weekly self-improvement skill curator is unchanged.

## References

- [[2026-07-16-cometline-agents.md]]
- [[2026-08-03-cometline-current-repo-update.md]]
