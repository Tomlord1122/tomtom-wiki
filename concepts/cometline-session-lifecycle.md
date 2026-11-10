# Cometline Session Lifecycle

Cometline's recent desktop work makes session state behave like a durable product feature instead of a byproduct of the current route. The main concerns are startup restoration, unused-session cleanup, unread background output, navigation history, and mini-window coordination.

## Startup restoration

`bootstrapHomeSession()` runs after the connection, workspace, and session list are ready:

1. Read the most recently visited session ID from localStorage.
2. Verify that the session still exists.
3. Navigate to that session when it exists.
4. Otherwise select the most recently updated session.
5. If no session exists, create a new chat.

The persisted key is `cometline:last-active-session-id`. `session-visit-history.svelte.ts` also maintains a bounded history of up to 50 visited sessions for back and forward navigation. Missing sessions are removed while traversing the history.

This keeps the app from treating the home route as a reset point. A restart returns the user to useful context rather than presenting an empty composer every time.

## Unused-session cleanup

The backend marks newly created sessions as disposable through `sessions.is_disposable`. Existing sessions are conservatively migrated to non-disposable. On startup, `PruneUnusedUserSessions()` removes top-level user sessions that were created but never changed or given a transcript. Cleared conversations are retained.

The cleanup runs in `cometmind/cmd/serve.go` before the server begins handling requests. This prevents a never-started New Chat row from polluting the initial session list while preserving deliberate empty or cleared sessions.

The database schema is currently version 28. The v27 to v28 migration adds `is_disposable` and marks pre-existing sessions as preserved.

## Unread background output

A session can receive assistant output while the user is viewing another session or another window. `unread-session-output.svelte.ts` tracks these session IDs in localStorage under `cometline.unread-session-output.v1`.

- Background output marks the target session unread.
- Opening or switching to the session marks it read.
- The store removes IDs that are no longer in the session list.
- `window-sync.ts` broadcasts `session-output-unread` so the full window and mini window stay aligned.
- Sidebar rows render the unread state.

This is intentionally UI-local state. It does not add unread columns to the server database because the signal describes the user's current viewing state, not durable conversation data.

## Navigation and focus

Navigation and new-chat actions now request composer focus explicitly after the destination is ready. This avoids focus depending on incidental component timing, especially after sidebar navigation, session creation, or mini-window transitions. The behavior is covered by action and mini-window tests.

## Mini-window coordination

The mini window reuses an existing session when possible and creates a persisted session only when needed. Its loading path coordinates session identity before rendering the chat surface, reducing flashes of the wrong session or an unnecessary new-chat state. Cross-workspace navigation is handled by the mini session helpers rather than by individual UI components.

## Design pattern

The recent changes establish a useful boundary:

- The backend owns durable session identity, transcript cleanup, and migration semantics.
- The renderer owns viewing state, navigation history, unread indicators, and focus requests.
- Window synchronization connects multiple renderer surfaces without turning ephemeral UI state into database state.

That separation keeps startup cleanup safe and keeps unread status responsive without complicating the session API.

## References

- [[2026-08-09-cometline-session-lifecycle-update.md]]
- [[cometline-frontend]]
- [[cometline-desktop-shell]]
- [[cometline-database]]
- [[cometline-project]]
