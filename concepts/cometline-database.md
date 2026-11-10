# Cometline Database Schema

The SQLite database at `~/.cometmind/cometmind.db` managed through `modernc.org/sqlite` (pure Go, no CGO) and sqlc-generated code.

## Schema Version

Current: **v28**. Migrations are incremental via `alterStatements` in `cometmind/internal/db/migrate.go`.

## Tables

### Core

| Table | Purpose |
|---|---|
| `workspaces` | Registered project directories (`id`, `name`, `path`) |
| `sessions` | Conversation threads scoped to a workspace, with status (`active`/`archived`), origin (`user`/`autonomy`/`inbox`), disposable-new-chat marker, delegation state, parent session forked-from links, context compaction state |
| `messages` | Individual turns within a session (`user`, `assistant`, `tool_result`, `system`). Stores `reasoning_content` as JSON array, `injected_memories`, `token_count` |
| `tool_calls` | Per-message tool invocations (`tool_name`, `arguments`, `result`, `duration_ms`, `exit_code`) |
| `assistant_provider_states` | Per-message provider/model tracking for which LLM generated each message |

### Memory

| Table | Purpose |
|---|---|
| `memories` | Semantic memories with `scope` (global/workspace), `kind` (fact/preference/project/task_outcome), `embedding` (BLOB), `base_weight`, `pinned`, `archived` |
| `memories_fts` | FTS5 virtual table on memory content for full-text search |
| `memory_events` | Audit log of memory CRUD actions |
| `memory_reembed_jobs` | Durable background re-embedding jobs for embedding model migration (`id`, `status`, `from_model`, `to_provider`, `to_model`, `to_base_url`, `total`, `completed`, `cursor_id`, `error`) |

### Jobs & Scheduling

| Table | Purpose |
|---|---|
| `jobs` | Work queue items with status (`todo`/`ongoing`/`done`/`blocked`), workspace, lease (session + expiry), retry (`failure_count`, `next_retry_at`), source platform |
| `scheduled_jobs` | Recurring/one-shot schedules with `cron_expr` or `run_at`, `enabled` flag, `next_run_at` |
| `job_events` | Immutable job history (`action`, `detail`, `actor_session_id`) |

### Gateway

| Table | Purpose |
|---|---|
| `gateway_sessions` | Discord platform sessions mapped to CometMind sessions (`platform`, `platform_user_id`, `platform_channel_id`, `thread_id`) |

### Inbox

| Table | Purpose |
|---|---|
| `inbox_messages` | Agent-to-user notifications with `title`, `body`, `status` (open/archived), `user_reply`, `job_id`/`session_id` links |

### Capability Tracking

| Table | Purpose |
|---|---|
| `model_capability_negatives` | Negative cache for model features that failed (`provider_id`, `model_id`, `feature`, `expires_at`) to avoid retrying unsupported operations |

## Key Indices

- `idx_sessions_workspace` — workspace → sessions
- `idx_sessions_updated` — recent sessions
- `idx_messages_session` — session timeline
- `idx_tool_calls_message` — tool calls per message
- `idx_jobs_status_updated` — job queue ordered by priority
- `idx_scheduled_jobs_due` — due schedules
- `idx_memories_active` — non-archived memories by scope
- `idx_memories_fts` — full-text memory search

## Code Generation

SQL → Go via `sqlc`:
- Schema: `cometmind/internal/db/schema.sql`
- Queries: `cometmind/internal/db/queries/*.sql`
- Output: `cometmind/internal/db/*.sql.go`, `db.go`, `models.go`

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
