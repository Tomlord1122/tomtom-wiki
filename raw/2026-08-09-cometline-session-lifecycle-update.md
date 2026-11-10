---
title: Cometline session lifecycle and desktop reliability update
source_url: git repository at @cometline, commits f7e1ee4..HEAD
captured_at: 2026-08-09
---

# Cometline session lifecycle and desktop reliability update

Inspected repository `@cometline/` at revision `26c9d92` on 2026-08-09. The previous wiki platform snapshot was revision `f7e1ee4` on 2026-08-03. Recent changes are concentrated in session lifecycle, mini-window behavior, unread output state, startup cleanup, and a few reliability fixes.

## Recent commits

- `49e88a0` restore last session on startup.
- `2bda075` prune unused sessions on startup.
- `40c2708` add unread session indicators.
- `2f731a1`, `c68b324`, `b5c76a3`, `3d33bb1` make composer focus explicit after session navigation and creation.
- `222f77c` polish mini-window loading.
- `3764fbd` fix backup logic.
- `6f64fb6` fix UI jitter.
- `65613f0` fix icon layout overlap.
- `4b4de55` fix DeepSeek model replay error.
- `de34e15` dismiss selection-to-chat popup on outside click.

## Session startup and cleanup

`cometline/src/lib/actions/bootstrap-home-session.ts` now restores the most recently visited existing session when the home route is ready. The last session ID is persisted in localStorage under `cometline:last-active-session-id`. A bounded in-memory visit history supports back/forward navigation and drops missing sessions. If there is no valid recent session, the newest session is selected; if no sessions exist, New Chat creates one.

`cometmind/internal/session/service.go` adds `PruneUnusedUserSessions`. It deletes top-level user sessions that were created but never changed or given a transcript, while retaining cleared conversations. `cometmind/cmd/serve.go` runs this before serving requests so a never-started New Chat row does not appear at startup.

Database schema is now version 28. Migration v27 to v28 adds `sessions.is_disposable`, defaults it to disposable for new rows, and marks existing rows as non-disposable. The field allows startup cleanup to distinguish unused new chats from preserved sessions.

## Unread output

`cometline/src/lib/stores/unread-session-output.svelte.ts` persists unread session IDs in localStorage under `cometline.unread-session-output.v1`. Background stream output marks a session unread; opening the session marks it read. The state is pruned against the current session list and synchronized between windows through `window-sync.ts` using the `session-output-unread` message.

`SessionRow.svelte`, `MiniSessionSidebar.svelte`, and related shell components display unread state. The store is covered by chat and store tests.

## Navigation and mini window

Session navigation and new-chat actions now issue explicit composer focus requests after navigation or session creation. Mini-window session helpers reuse or create a persisted session, navigate across workspaces, and coordinate loading state so the mini surface does not briefly render the wrong session or an unnecessary new-chat state.

## Other reliability changes

The recent range also contains backup behavior fixes, UI jitter and icon-overlap fixes, a DeepSeek reasoning replay fix, and outside-click dismissal for the selection-to-chat popup. These are product reliability improvements rather than new architectural subsystems.

## Relevant code paths

- `cometline/src/lib/actions/bootstrap-home-session.ts`
- `cometline/src/lib/stores/session-visit-history.svelte.ts`
- `cometline/src/lib/stores/unread-session-output.svelte.ts`
- `cometline/src/lib/stores/session.svelte.ts`
- `cometline/src/lib/actions/navigate-to-session.ts`
- `cometline/src/lib/actions/new-chat.ts`
- `cometline/src/lib/mini-window-session.ts`
- `cometmind/internal/session/service.go`
- `cometmind/cmd/serve.go`
- `cometmind/internal/db/migrate.go`
- `cometmind/internal/db/schema.sql`
