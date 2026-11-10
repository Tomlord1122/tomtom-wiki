# Cometline Desktop Shell

The SvelteKit + Electron frontend that provides the native desktop chat UI, settings management, and sidecar lifecycle for Cometline.

## Architecture

```
Electron main process
  → spawns cometmind sidecar binary
  → persists desktop provider settings
  → exposes OS/native capabilities over preload IPC

SvelteKit renderer
  → HTTP/SSE to cometmind at 127.0.0.1:7700
  → reactive stores for chat, session, model, settings
  → controller pattern for complex UI features
```

## Route Surfaces

| Route | Surface |
|---|---|
| `/` | New chat hero composer |
| `/session/[id]` | Full chat thread |
| `/jobs` | Jobs board and job detail drawer |
| `/skill-drafts` | Skill draft review and promotion |
| `/settings` | Direct settings route (outside AppShell) |
| `/mini` and `/mini/session/[id]` | Compact mini-window chat |

## Mini Window

The mini window (`/mini`) is a compact, always-on-top chat surface. Key components:

- **MiniShell.svelte** — keyboard-driven shell with sidebar toggle, new chat
- **MiniSessionSidebar.svelte** — slide-out drawer with search, pin, delete, workspace-grouped session list
- **mini-window-session.ts** — session lifecycle: `ensureMiniWindowSession` (reuse or create), `navigateMiniToSession` (cross-workspace navigation), `createMiniWindowSession` (new persisted session)
- **Session expiration**: mini-window state persists a `MiniWindowState` (sessionId, lastActiveAt, inactivityTimeoutMinutes); expired sessions trigger new session creation
- **Cross-window state**: unread session output is synchronized through `window-sync.ts`, while the mini sidebar displays the same session-level unread indicators as the main window
- **Loading coordination**: session identity is established before rendering the mini chat surface, avoiding a flash of the wrong session or an unnecessary new chat

## Workspace Panels

The main window supports two workspace panel surfaces docked below the chat area:

| Panel | Shortcut | Purpose |
|---|---|---|
| Web Panel | Cmd+O | File tree, previews, wikilinks, backlinks, web context |
| Terminal Panel | Cmd+J | In-app terminal via xterm.js + Electron PTY per session |

See [[cometline-web-panel-and-wiki]] and [[cometline-terminal-panel]].

## Frontend Patterns

### File Roles

| Extension | Purpose |
|---|---|
| `.svelte` | Markup, bindings, `$effect` wiring only |
| `.svelte.ts` | Reactive controllers (`$state`, `$derived`, callbacks) |
| `.ts` | Pure functions — unit-testable, no Svelte imports |

### Controller Pattern

Heavy UI features use `create…Controller(deps)` factories with getter callbacks:

```typescript
export function createThreadScroll(deps: { getScroller: () => HTMLElement | null }) {
    // $state + methods
    return { scrollToBottom, setScroller };
}
```

### Chat Thread Layout

```
ChatThread.svelte          — thin orchestrator + {#each} dispatch
├── createFoldController   — expand/collapse state
├── createThreadScroll     — scroll anchoring
├── createThreadClocks     — copy feedback, memory cycle tick
├── thread-visibility.ts   — pure show/hide predicates
└── Row components         — UserMessageRow, AssistantMessageRow, …
```

## Error Taxonomy

| Level | When | UI |
|---|---|---|
| Fatal | CometMind unreachable | `RuntimeOverlay` — blocks interaction, retry |
| Route | SvelteKit load failure | `+error.svelte` |
| Recoverable | Send failed, session load failed | `ErrorBanner` inline |
| Inline | Field validation | Adjacent to control |

## Styling

- CSS custom properties (`var(--*)`) tokens in `app.css`
- Semantic status colors: `--status-success`, `--status-error`
- No hardcoded hex in components — add tokens for new semantic colors

## Sidecar Lifecycle

The Electron main process manages the cometmind binary:
1. Spawn as child process on app launch
2. Monitor health on port 7700
3. Persist settings to `~/.cometmind/cometline-settings.json` and `cometline-desktop.json`
4. Restart on settings changes that require process-level bind changes (host/port)

## Git graph expand and historical diff

An expanded commit card must stay in document flow and push later rows. Measure its `clientHeight` and add that height to later rail coordinates and the overlay SVG height. A panel background plus z-index only hides the line; it does not uncover the next row. Declare the height state and the expansion derived before any derived that reads them.

A file in that card opens the commit diff, not the worktree diff. Keep path and SHA separate. Store the SHA in the surface that renders the diff, because the graph browser and the diff view are siblings. Add an optional `sha` query to the existing file-diff operation, regenerate the client, and disable worktree mutations while it is set.

Each graph text row is the pane width, not the message width. `width: max-content` on the row, the uncommitted row, or the detail slot lets a long subject or chip slide the SHA off the panel edge. Pin those to `width: 100%`, give the subject track `minmax(0, 1fr)`, and ellipsize chips and the subject. Do not wrap chips; the rail height stays 28px. Landed on `dev` as `2e966793` (`fix(workspace): pin commit shas to the graph panel edge`).

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-architecture]]
- [[cometline-project]]
