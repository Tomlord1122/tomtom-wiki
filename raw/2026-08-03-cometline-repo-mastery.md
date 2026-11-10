---
title: Cometline repository mastery audit and learning path
source_url: file:///Users/tomlord/workspace/personal-project/cometline
source_revision: f7e1ee4 (plus fix/selection-add-chat-dismissal @ de34e15)
captured_at: 2026-08-03
---

# Captured source

Comprehensive read of the Cometline monorepo documentation and learning guides, done to refresh wiki knowledge and build a teaching path.

## Canonical docs read

- `ARCHITECTURE.md` — system overview, module responsibilities, load-bearing contracts.
- `ARCHITECTURE_GUIDE.md` — contributor map with line references.
- `docs/MODULE_GUIDE.md` — ownership rules and design rules per module.
- `docs/SETTINGS_AND_PERSISTENCE.md` — three persistence modes, draft model, dirty-state rules.
- `docs/learning/00-README.md` through `10-development-guide.md` — the full learning path.
- `cometline/docs/COMETLINE_ARCHITECTURE.md` — desktop architecture details.
- `cometline/docs/FRONTEND_PATTERNS.md` — renderer patterns.
- `cometline/docs/postmortem/README.md` — UI bug history index.

## Key facts confirmed during audit

- No root `go.work`; run Go commands from `comet-sdk/` or `cometmind/`.
- Dependency direction is strict: `cometline → cometmind → comet-sdk`.
- CometMind API binds to `127.0.0.1:7700` only; CORS allows Vite dev, localhost, `app://`, `file://`, empty origin.
- Sidecar stop must wait for process exit before restart, or port 7700 and SQLite WAL lock collide.
- Settings live in `~/.cometmind/cometline-settings.json` (runtime, mode 0600) and `~/.cometmind/cometline-desktop.json` (desktop keys). Electron merges for UI and splits on write. `config.toml` is legacy fallback only.
- Agent loop: model call → persist turn → execute tools → continue, up to 100 work steps plus final answer (budget changed from 50 to 100 in recent commits; the July wiki pages said 50).
- `agent.Runner` depends on a `TurnStore` interface, not concrete SQLite, making the loop testable.
- One in-flight run per session enforced by `server/run_manager.go`; rapid-fire messages are serialized, not dropped.
- Tool surfaces are a capability policy: `ParentSurface` (main agent, full), `ResearchSurface` (read + skills), `CodingSurface` (read + edit + run + skills).
- Provider factory dispatches `opencode-go` models by models.dev protocol metadata: `@ai-sdk/openai` → OpenAI Responses (`openairesponses`), `@ai-sdk/anthropic` → Anthropic Messages, default → Chat Completions.
- Shared `responsesproto` package in comet-sdk is reused by Codex and OpenCode Go providers.
- OpenAPI `cometmind/openapi.yaml` generates the TypeScript client (`@hey-api/openapi-ts`) and Go types (`oapi-codegen`).
- SSE contract source of truth is `cometmind/internal/event/event.go`, mirrored in OpenAPI and `cometline/src/lib/types.ts`.
- SQLite schema lives in `cometmind/internal/db/schema.sql`; migrations tracked via `schemaVersion` in `migrate.go` (current 26). Never hand-edit generated sqlc files.
- Electron BrowserWindow: `contextIsolation: true`, `nodeIntegration: false`, preload-only bridge (`window.electronAPI`).
- Production packaged app uses `app://bundle` custom protocol with SPA fallback; without it session route reload 404s.
- The learning docs were built from the GitNexus knowledge graph for `cometline-release` (10,327 symbols / 28,474 relationships / 300 flows at build time; the index in AGENTS.md now reports 13,650 symbols / 38,598 relationships).

## Known drift between repo docs and older wiki pages

- `cometline-architecture` wiki page says agent loop "up to 50 steps"; current docs say 100 work steps plus final answer.
- Provider list in older wiki pages omits `openairesponses` and the `opencode-go` protocol dispatch.
- Current frontend uses Svelte 5 runes (`$state`, `$derived`, `$effect`) and module-level store singletons, not the older Svelte store pattern some wiki pages describe.
- Tool surface model is capability-based (Parent/Research/Coding) rather than a fixed per-surface hand-picked tool list.
- `docs/learning/05` lists `schemaVersion 26`; older wiki pages may cite older versions.
