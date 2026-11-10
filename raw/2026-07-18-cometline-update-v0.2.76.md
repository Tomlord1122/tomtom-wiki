---
title: Cometline v0.2.76 update — inbox, capability learning, web panel file tree, and UX improvements
source_url: file://personal-project/cometline (git range a4ccb81..b048cc8)
captured_at: 2026-07-18
---

# Cometline v0.2.76 Update Capture

Repository state: `b048cc8` (`v0.2.76`), merged PR #54 (`feat/web-panel-file-tree`).

## Commit timeline

- `6f37cd6` — add inbox OpenAPI contract and generated clients
- `02f4986` — implement inbox persistence service
- `cae03b4` — add `leave_inbox_message` tool
- `848fd0b` — process archived inbox replies
- `148fd66` — wire inbox API, retention, and background worker
- `9667235` — add inbox modal and skill drafts badge
- `4dc13aa` — add panel shortcuts for jobs, drafts, and inbox
- `c2ec847` — harden backups and wiki paths
- `ed1b8e3` — add learned model capability resolver
- `2d0f234` — increase stream idle timeout to 10 minutes
- `3d04764` — handle incomplete tool calls and improve user nudges
- `ecf153a` — integrate jobs indicator store and sidebar notifications
- `6757850`, `345bb1b` — improve mini-window behavior
- `6147fa3` — add wiki backlinks and path-only file references
- `8f3f53c` — add web panel tree, wikilink, and snippet helpers
- `d1fc679` — accumulate panel contexts and panel history in shell
- `b94a1fb` — wire file tree, preview snippets, and `@dir` mentions
- `3ff780d` — fix lint warning
- `b048cc8` — merge web panel file tree feature

## Main changes

### Inbox

Added persistent inbox messages, `leave_inbox_message`, REST API, background processing of archived replies, retention integration, desktop inbox drawer, notifications, and skill-draft badges. Inbox replies can be processed and internalized by the runtime.

### Learned model capability compatibility

Added `cometmind/internal/modelcompat/Resolver`. When a provider explicitly rejects an optional capability, the runtime records a negative capability for the provider/endpoint/model/feature tuple for seven days. Future requests resolve a capability policy before sending. Expired negatives are cleaned up. This keeps provider-specific quirks out of the agent loop.

### Incomplete tool calls and nudges

The agent runner now handles incomplete tool calls more safely and emits user-facing nudges when a turn cannot proceed normally. New tests cover runner behavior and chat reducer rendering.

### Codex and SDK streaming

The SDK stream model, Codex conversion/client/stream handling, SSE scanner, and error surface were enhanced. Codex response validation and optional output handling received dedicated tests. Provider state is persisted per assistant message.

### Web Panel and file tree

The desktop shell now has a richer web/file panel experience:

- File tree browser with workspace files
- File preview and editor integration
- Preview snippets and directory mentions via `@dir`
- Wiki file indexing, wikilinks, and backlinks
- File/path references rendered as interactive mentions
- Link embed chips with favicons for bare HTTP(S) URLs
- Panel contexts and history accumulated in shell state
- Focus/navigation helpers and panel shortcuts

### Navigation and mini window

Added session visit history and forward/back navigation. Mini-window behavior now supports fullscreen and better button visibility, with spurious activation suppressed.

### Jobs and desktop UX

Added jobs indicator store, sidebar notifications, app toasts, close confirmation modal, improved inbox/job/skill-draft badges, and keyboard shortcuts.

### Backup and path hardening

Backup handling and wiki paths were hardened. The runtime now treats wiki paths and file references with more explicit safety boundaries.

## Contract surface

The OpenAPI spec now includes wiki endpoints:

- `listWikiFiles`
- `listWikiFileBacklinks`
- `readWikiFileContent`
- `writeWikiFileContent`

The current spec contains 78 operation IDs across health, models, workspaces, files, wiki, sessions, skills, MCP, memory, storage, jobs, scheduling, and inbox.
