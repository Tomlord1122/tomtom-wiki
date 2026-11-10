# Cometline Frontend Architecture

The SvelteKit + Electron desktop shell at `cometline/src/lib/`. Follows a strict separation: `.svelte` for markup, `.svelte.ts` for reactive controllers, `.ts` for pure functions.

## Store Layer

Cross-route reactive state in Svelte 5 runes (`$state`, `$derived`):

| Store | File | Holds |
|---|---|---|
| `chatStore` | `stores/chat.svelte.ts` | Active conversation messages, streaming state, tool call display |
| `sessionStore` | `stores/session.svelte.ts` | Session list, active session ID, creation/deletion |
| `modelStore` | `stores/model.svelte.ts` | Available models, selected model, provider state |
| `settingsStore` | `stores/settings.svelte.ts` | Provider configs, appearance, shortcuts, persona |
| `workspaceStore` | `stores/workspace.svelte.ts` | Active workspace, file index, registered workspaces |
| `shellStore` | `stores/shell.svelte.ts` | App shell state: sidebar, panels, route transitions |
| `jobStore` | `stores/job.svelte.ts` | Job queue, job detail, filtering |
| `terminalStore` | `stores/terminal.svelte.ts` | Per-session terminal instances, data streaming, IPC bridge |
| `sessionVisitHistory` | `stores/session-visit-history.svelte.ts` | Bounded back and forward history plus persisted last active session |
| `unreadSessionOutputStore` | `stores/unread-session-output.svelte.ts` | Unread background output, local persistence, and cross-window sync |

## Controller Pattern

Heavy features use `create…Controller(deps)` factories accepting getter callbacks:

```
createThreadScroll({ getScroller: () => HTMLElement | null })
createFoldController({ getMessages: () => Message[] })
createThreadClocks({ onMemoryCycle: () => void })
```

Controllers live in `.svelte.ts` files alongside their `.svelte` shell. The `.svelte` file wires props → mounts controller → renders child components.

## Chat Thread Architecture

```
ChatThread.svelte              — thin orchestrator + {#each} dispatch
├── createFoldController       — expand/collapse message groups
├── createThreadScroll         — scroll anchoring, auto-scroll
├── createThreadClocks         — copy feedback timing, memory cycle tick
├── thread-visibility.ts       — pure show/hide predicates
├── subagent-display.ts        — merge subagent results into display
├── mcp-tool-display.ts        — map MCP tool names to readable labels
├── reasoning.ts               — reasoning block parsing and rendering
└── Row components             — UserMessageRow, AssistantMessageRow, ToolCallRow, …
```

### Chat Turn Dispatch (`actions/start-chat.ts`)

The SSE lifecycle manager:
1. Builds the API request (messages + model + workspace)
2. Opens SSE connection to `POST /api/v1/sessions/{id}/messages`
3. Dispatches events to chat reducer (`reducers/chat.ts`)
4. Handles stream errors, reconnection, cancellation

## Route Architecture

| Route | Surface | Key Components |
|---|---|---|
| `/` | Hero composer | `Composer.svelte` with model picker, workspace context |
| `/session/[id]` | Full chat | `ChatThread.svelte` + `ChatView.svelte` |
| `/jobs` | Jobs board | Kanban columns + job detail drawer |
| `/skill-drafts` | Draft review | Draft list + diff view + promote/reject |
| `/settings` | Settings (outside AppShell) | Tabbed panels: Providers, Models, Appearance, Shortcuts, Memory, MCP, About |
| `/mini` | Mini window chat | Compact ChatThread variant |
| `/mini/session/[id]` | Mini session | Same, scoped to session |

## Session Lifecycle

`actions/bootstrap-home-session.ts` restores the most recently visited existing session after the connection, workspace, and session list are ready. It falls back to the newest session, or creates a new chat when the list is empty. `session-visit-history.svelte.ts` persists the last active ID and maintains a bounded navigation history.

Streaming output that arrives while another session is active is tracked by `unreadSessionOutputStore`. The store persists unread IDs in localStorage, marks them read when opened, prunes deleted sessions, and broadcasts changes through `window-sync.ts` so the main and mini windows agree. Session rows render the indicator.

Navigation and new-chat actions issue explicit composer focus requests after a destination is ready. This makes focus deterministic across sidebar navigation, session creation, and mini-window transitions.

See [[cometline-session-lifecycle]] for the backend and renderer boundary.

## Ollama Integration (Frontend)

Ollama support is spread across three layers:

- **`lib/ollama/catalog.ts`** — static versioned model catalog with capabilities, size estimates, architecture notes
- **`lib/ollama/client.ts`** — renderer-side IPC wrappers: health check, model listing, pull with progress, cancel
- **`lib/ollama/url.ts`** — URL normalization: native base ↔ chat base conversion, loopback detection
- **`components/settings/OllamaProviderPanel.svelte`** — full settings panel with catalog cards, custom pull, health display
- **Setup wizard** — onboarding flow: install Ollama → pull models → assign roles

See [[cometline-ollama-integration]] for the full architecture.

## Terminal Panel (Frontend)

- **`components/TerminalPanel.svelte`** — panel shell with start/stop, surface switcher, toolbar
- **`components/TerminalInstance.svelte`** — xterm.js Terminal + FitAddon, selection-to-context
- **`stores/terminal.svelte.ts`** — reactive store bridging IPC and xterm.js subscribers

Terminal text selection is preserved across re-renders via `selectionState` tracking in `TerminalInstance`, allowing the floating "Add to context" button to survive scroll and resize events.

See [[cometline-terminal-panel]] for details.

## First-Turn Flight Animation

The composer-to-thread transition uses a "flight" animation: the user's message bubble and the assistant's avatar animate from the composer area to their final positions in the chat thread. This creates a physical, spatial connection between input and response.

| File | Purpose |
|---|---|
| `first-turn-flight.ts` | Core utilities: `rectStyle`, `rectTransform`, `measureStableRect`, `animateElementToRect`, `waitForAnimationEnd`, `afterPaint` |
| `FirstTurnFlight.svelte` | Orchestrator: avatar flight-in + triggers `UserBubbleFlight` run |
| `UserBubbleFlight.svelte` | User message "particle" that flies from composer rect to thread position via CSS `@keyframes` using `--flight-x`/`--flight-y` custom properties |

**Key behaviors:**
- Measures the composer textarea and target thread position via `getBoundingClientRect`
- Waits for layout stability (`measureStableRect`, 40-frame max) before animating
- Duration: 560ms, easing: `cubic-bezier(0.22, 1, 0.36, 1)`
- Respects `prefers-reduced-motion: reduce` — disables animation entirely
- Flight state is preserved across re-renders (message staging → flight → reveal pipeline)
- Scroll anchoring is disabled during flight to prevent jarring jumps

## Settings System

`settings/schema.ts` defines the `ProviderSettings` type with normalization. The settings pipeline:

1. **Desktop**: Electron main persists `cometline-settings.json` + `cometline-desktop.json`
2. **IPC**: Main ↔ renderer via `cometline:*` channels
3. **Draft**: `settings-draft.ts` tracks unsaved changes
4. **Save**: `settings-save.ts` computes `RuntimeApplyAction` — in-place reload for most changes, gateway restart for Discord, full restart only for host/port

The model-role panel now treats `defaultProviderId` + `defaultModelId` as the global Default pair. Helper-role pins are optional and are valid only as complete provider/model pairs; incomplete pins fall back to Default. The UI lists explicitly enabled models and removes pins aimed at disabled or model-less providers. `activeProviderId` is legacy input kept for migration, not a setting written back to disk. See [[cometline-model-roles]].

## Thinking and transition feedback

The chat surface exposes phase-specific feedback rather than one generic spinner:

- `orbit` — thinking/preparation;
- `eclipse` — tool execution;
- `nova` — composing/final response.

`sticky-thinking-variant` keeps the selected visual stable across fast phase changes. The first-turn flight pipeline also preserves staged message state across re-renders, disables scroll anchoring while the message travels from the composer to the thread, and restores it afterward.

## Shortcut tooltips

The desktop shell uses a portal-backed `Tooltip.svelte` with viewport-aware positioning. `shortcut-tooltip.ts` maps chrome actions to live labels and keyboard shortcuts, keeping tooltips from being clipped by sidebars, terminals, or web panels. Pure positioning and shortcut resolution have focused tests.

## Error Handling

| Level | Trigger | UI Component |
|---|---|---|
| Fatal | CometMind unreachable | `RuntimeOverlay` (blocks interaction, `role="alert"`) |
| Route | SvelteKit load error | `+error.svelte` |
| Recoverable | Send fail, session load fail | `ErrorBanner` (dismissible, `role="alert"`) |
| Inline | Field validation | Adjacent error text |

## Styling

- CSS custom properties from `app.css` (`var(--*)`)
- Semantic tokens: `--status-success`, `--status-error`
- Shared chrome: `ThreadRow.svelte` for chat rows
- No hardcoded hex — new semantic colors get new tokens

## Key Frontend Modules

| Module | Purpose |
|---|---|
| `client/cometmind.ts` | HTTP/SSE client wrapping all CometMind API calls |
| `reducers/chat.ts` | SSE event → UI state reducer |
| `workspace/` | File index, preview overlay, CodeMirror integration |
| `wiki/` | Cached wiki file index for wikilinks and file navigation |
| `skills/` | Slash-command parsing, skill registry, and draft indicators |
| `jobs/` | Job formatting, grouping, proposal parsing, and sidebar indicator |
| `inbox/` | Inbox link availability and notification behavior |
| `stores/inbox.svelte.ts` | Inbox messages, summary, replies, dismissals |
| `stores/app-toasts.svelte.ts` | App-level toast notifications |
| `stores/session-visit-history.svelte.ts` | Session back/forward navigation history |
| `components/FileTreeBrowser.svelte` | Workspace file tree |
| `components/InboxDrawer.svelte` | Asynchronous inbox notification drawer |
| `components/WebPanel.svelte` | Web/file/wiki panel surface |
| `personas/` | Minako ↔ Souma switching: avatar, icon, SOUL prompt |
| `markdown/` | Markdown → HTML rendering for assistant messages |
| `actions/` | Svelte actions wiring UI events to store/API calls |
| `conversation/` | Pure logic: fold, scroll, reasoning, subagent display |

## Testing

- **Pure logic**: `*.test.ts` in `node` environment (Vitest)
- **Components**: `*.svelte.test.ts` in `jsdom` with `@testing-library/svelte`
- **E2E**: Playwright available

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-architecture]]
- [[cometline-desktop-shell]]
- [[cometline-project]]
