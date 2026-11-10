# Cometline Inbox System

The inbox became a complete asynchronous communication surface in the v0.2.76 update: runtime persistence, agent tool, background processing, API, and desktop drawer.

## Purpose

Chat is synchronous; the inbox is for results and requests that outlive a single turn. The runtime can notify the user when an autonomous job completes, when delegated work needs attention, or when a background process has something worth surfacing.

## Runtime Flow

```text
agent/tool/background worker
  → leave_inbox_message
  → inbox_messages SQLite row
  → CometMind inbox API
  → desktop inbox drawer / notification
  → user replies or dismisses
  → inbox worker processes reply
```

## Persistence

`inbox_messages` stores title, body, workspace, related job/session, open/archived state, reply text, processing timestamps, retry attempts, and archive reason.

Archive reasons distinguish a user reply from a dismissal:

- `replied`
- `dismissed`

## API

The OpenAPI contract and generated clients expose:

- `listInboxMessages`
- `getInboxSummary`
- `getInboxMessage`
- `replyInboxMessage`
- `dismissInboxMessage`

The server handlers live in `cometmind/server/inbox_handlers.go`.

## Background Worker

`inboxworker` processes archived replies. A reply can become input to the agent's durable knowledge flow, including internalization as semantic memory. Processing attempts and errors are persisted so failures can be retried and diagnosed.

Inbox retention is integrated with the runtime cleanup system.

## Desktop Experience

`InboxDrawer.svelte` gives the user a dedicated notification surface. The shell also adds:

- Inbox badge/summary indicators
- Deep links to related jobs and sessions when available
- Keyboard/panel shortcut access
- Toasts for new or important events

Skill drafts and jobs expose similar indicator patterns, creating a consistent asynchronous work center.

## Relationship to Jobs and Autonomy

The inbox is the output channel for work that happens away from the active chat. Jobs provide durable execution; autonomy performs work; the inbox delivers the result back to the user.

## References

- [[2026-07-18-cometline-update-v0.2.76.md]]
- [[cometline-inbox-gateway]]
- [[cometline-jobs-scheduling]]
- [[cometline-semantic-memory]]
- [[cometline-api]]
- [[cometline-project]]
