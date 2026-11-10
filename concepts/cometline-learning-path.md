# Cometline Learning Path

The reading order to go from "new to Cometline" to "can reason about the whole system" using the repo's own `docs/learning/` series, now also summarized in the wiki.

## Why this order

Each guide assumes the previous one. The order builds the mental model bottom-up from what the product is, to the boundaries, to the flows, to each layer's internals, then to the contracts and the day-to-day development loop.

| Step | Doc | What you learn |
|---|---|---|
| 1 | `docs/learning/01-nutshell.md` | What Cometline is, the three modules, one message end to end |
| 2 | `docs/learning/02-architecture.md` | Ownership rules, dependency direction, load-bearing contracts |
| 3 | `docs/learning/03-data-flows.md` | Startup, first message, agent loop, settings save/reload, MCP, jobs, packaging |
| 4 | `docs/learning/04-comet-sdk.md` | Provider interface, streaming, retries, tool-call assembly |
| 5 | `docs/learning/05-cometmind-runtime.md` | Agent runner, sessions, SQLite, HTTP/SSE server |
| 6 | `docs/learning/06-cometmind-features.md` | Memory, MCP, coding harness, Discord, skills, jobs |
| 7 | `docs/learning/07-cometline-desktop.md` | Electron main, sidecar lifecycle, IPC, settings persistence |
| 8 | `docs/learning/08-cometline-frontend.md` | SvelteKit routes, stores, SSE reducer, chat/jobs/settings UI |
| 9 | `docs/learning/09-contracts-codegen.md` | OpenAPI, SSE events, sqlc, generated clients |
| 10 | `docs/learning/10-development-guide.md` | Build/test commands, extension recipes, verification |

## Companion docs

- `ARCHITECTURE.md` — system overview (source of truth).
- `ARCHITECTURE_GUIDE.md` — contributor map with line references.
- `docs/MODULE_GUIDE.md` — module ownership checklists for agents.
- `docs/SETTINGS_AND_PERSISTENCE.md` — settings modal draft/save rules.
- `docs/FRONTEND_DESIGN_SYSTEM.md` — visual tokens and styling.
- `cometline/docs/postmortem/README.md` — UI bug history index; read before touching `chat.svelte.ts`, `reducers/chat.ts`, `ChatView.svelte`, `Composer.svelte`, `slash-commands.ts`, settings, API client, or the OpenAPI/SSE contracts.

## Study approach

- Read 01 to 03 for orientation first; they are the shortest and give the whole picture.
- Then read 04 and 05 together with the actual code open: `comet-sdk/llm/stream.go` and `cometmind/internal/agent/runner.go`.
- 06 is wide but each section is independent; pick by feature interest (memory, MCP, jobs, ACP).
- 07 and 08 matter most when touching Electron or SvelteKit surfaces.
- 09 and 10 are reference material to return to whenever changing contracts or running builds.

## Related wiki pages

- [[cometline-project]] — product entity page.
- [[cometline-architecture]] — the three-layer pattern.
- [[cometline-agent-runtime]] — the agent loop.
- [[cometline-sdk-internals]] — provider normalization.
- [[cometline-database]] — SQLite schema.
- [[cometline-api]] — OpenAPI contract and SSE events.
- [[cometline-frontend]] — renderer architecture.
- [[cometline-desktop-shell]] — Electron shell.

## References

- [[2026-08-03-cometline-repo-mastery.md]]
- [[cometline-project]]
