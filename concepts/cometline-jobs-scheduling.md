# Cometline Jobs & Scheduling

Cometline's durable work queue and scheduling system that lets tasks outlive a single chat turn.

## Architecture

Three subsystems in `cometmind/internal/`:

- **`jobs/`** — Immediate work queue with job lifecycle (todo → ongoing → done/blocked)
- **`scheduler/`** — Scheduled job materialization from cron expressions or one-shot timestamps
- **`autonomy/`** — Autonomous job worker that can claim and execute jobs without user prompting

## Job Lifecycle

1. **Propose** — Agent or user describes work; job is proposed for confirmation
2. **Create** — Job lands in the todo queue with workspace path and definition of done
3. **Claim** — A session claims the job (via `claim_job` tool or jobs board UI)
4. **Execute** — Progress updates via `update_job` at each milestone
5. **Complete** — Marked done with final progress summary
6. **Release** — Returned to todo if abandoned

Jobs can also be blocked, unblocked, archived, and inspected via event history.

## Scheduled Jobs

Scheduled jobs are separate from immediate jobs:
- **One-shot**: `run_at` (Unix ms) or `run_at_iso` (RFC3339 with timezone)
- **Recurring**: `cron_expr` (standard 5-field cron)

When due, CometMind materializes a normal queue job automatically. Schedules can be paused (`enabled=false`), updated, or deleted without affecting already-materialized jobs.

## Jobs Board UI

The desktop shell has a dedicated `/jobs` route with:
- Kanban-style board (todo/ongoing/done/blocked)
- Job detail drawer with event history
- Create, edit, archive, unblock actions
- Job inspection with runtime lease info

## Autonomous Execution

When autonomy is enabled, CometMind's autonomous job worker can:
- Scan the todo queue
- Claim eligible jobs
- Execute them in a new session
- Report results via inbox messages

## Tool Interface

Agent tools for job management:
- `propose_job` — propose work for user confirmation
- `create_job` — create immediate todo job
- `claim_job` — claim a job for current session
- `update_job` — report progress
- `complete_job` — mark as done
- `release_job` — return to todo
- `list_jobs` — query job queue
- `create_scheduled_job` — schedule future/recurring work
- `list_scheduled_jobs` — inspect schedules

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
