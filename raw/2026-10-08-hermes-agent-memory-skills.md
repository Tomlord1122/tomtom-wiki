---
title: Hermes Agent memory, skills, and background review
source_url: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/
captured_at: 2026-10-08
---

Compiled from public pages fetched on 2026-10-08. Not a verbatim dump. Official docs were truncated mid-page; GitHub issues are maintainer-facing reports of current main, not product docs. The fan site and Vectorize article disagree with the official docs and those issues on a few points; disagreements are marked.

## Sources

- Official persistent memory: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/
- Official curator: https://hermes-agent.nousresearch.com/docs/user-guide/features/curator
- Fan explainer (not affiliated with Nous Research), updated 2026-10-07: https://hermes-agent.ai/blog/hermes-agent-memory-system
- GitHub issue #42388 (closed as not planned, sweeper:implemented-on-main), opened 2026-06-08: https://github.com/NousResearch/hermes-agent/issues/42388
- GitHub issue #94226 (open), opened 2026-08-24, claims verification on main commit f14059f: https://github.com/NousResearch/hermes-agent/issues/94226
- Vectorize vendor article, 2026-03-31: https://vectorize.io/articles/hermes-agent-memory-explained

Not captured: Medium (403), Zhihu (403).

## What Hermes is

Nous Research's Hermes Agent. A same-named Trend Micro mail-security service is unrelated; that compiled page was removed on 2026-10-10.

## Durable memory (official)

Two files under `~/.hermes/memories/` (per profile: `~/.hermes/profiles/<name>/memories/`):

| File | Purpose | Limit |
|---|---|---|
| MEMORY.md | Environment facts, conventions, lessons | 2,200 chars (~800 tokens) |
| USER.md | Preferences, communication style, expectations | 1,375 chars (~500 tokens) |

Injected as a frozen snapshot at session start so the prefix cache stays stable. Writes persist immediately but do not appear in the system prompt until the next session. Tool responses show live state. Two agent processes must not share one Hermes home; shared memory goes through an external provider.

The agent writes via a memory tool: `add`, `replace`, `remove`. No `read` action. `replace`/`remove` match a unique substring (`old_text`); `replace` overwrites the whole entry. Exact duplicates are rejected. Entries are scanned for injection, credential exfiltration, SSH backdoors, and invisible Unicode before acceptance.

Memory does not auto-compact. An over-limit write returns an error with `current_entries` and the shortfall. The agent must retry as one `operations` batch that frees enough characters and adds the new entry. The limit is checked only on the batch result. Above ~80%, consolidate before adding. Typical size: 8–15 memory entries, 5–10 user entries.

Save: preferences, environment facts, corrections, conventions, completed-work diary lines, explicit "remember this". Skip: vague trivia, web-searchable facts, raw dumps, session ephemera, anything already in SOUL.md or AGENTS.md. A spoken "I'll remember that" is not a write. `.env` is not memory. A path needed on every run of a recurring task often belongs in a skill, not the 2,200-char budget.

`write_approval: true` stages non-CLI writes (messaging, scripts, background review) for `/memory pending` / `approve` / `reject`. Staged replace/remove pin the full target entry; if it changed, approval is refused. Disabling both `memory_enabled` and `user_profile_enabled` drops the built-in tool. `memory.provider` (Hindsight, Mem0, Honcho, …) is independent. Listing `memory` under `agent.disabled_toolsets` also hides provider tools.

Session search: all CLI and messaging sessions in `~/.hermes/state.db` with FTS5. Official docs say search returns actual messages, no LLM summarization, no truncation; the agent can scroll inside a found session. Memory is ~1,300 tokens always in prompt; session search is on demand (~20ms FTS5, ~1ms scroll).

`/journey` (`hermes journey`) plots saved skills and memory entries over time. Node ids are skill names or `memory:<source>:<index>:<fingerprint>`. Delete archives skills (restorable) and removes memory chunks. A skill appears once it was created in-profile, created by background review, or used once. Bundled and unused hand-written skills stay off the timeline.

Official "Memory Needs Session Boundaries": the learning loop of forget → recall from memory → search past sessions only fires when a session ends. Messaging chats are one continuous session across restarts. Without `/new`, compaction reruns over an ever-longer history and fresh memory stays invisible because of the frozen snapshot. CLI invocations are naturally new sessions.

## Three layers (fan site, 2026-10-07)

Durable facts (declarative, not a task log), skills (procedures loaded only when relevant), session search (recall). SOUL.md is personality, not memory. A document index is not built-in memory: keep the file and a retrieval procedure outside the char budget. Profiles separate personal, work, and public bots.

## Curator (official)

Background maintenance for agent-created skills in `~/.hermes/skills/`. Not a cron. Fires when `interval_hours` (default 168, 7 days) has passed and the agent has been idle `min_idle_hours` (default 2). Checked on CLI session start, gateway housekeeping, and the Desktop/`hermes serve` hourly timer. Spawns a background AIAgent fork with its own prompt cache; does not touch the active conversation. First observation seeds `last_run_at` and defers the first real pass by one full interval.

Two phases:

1. Deterministic, always on when curator is enabled: unused `stale_after_days` (14) → stale; unused `archive_after_days` (30) → `~/.hermes/skills/.archive/`. Never auto-deletes. Pinned skills and skills referenced by any cron job (including paused) are skipped. `use_count == 0` is not archived until at least `stale_after_days` old.
2. LLM consolidation, off by default (`curator.consolidate: false`). Opt-in because it costs aux-model tokens (a sweep is described as typically 50–100 API calls) and rewrites the library. Aux slot is `auxiliary.curator`. The fork may keep, patch, merge into class-level umbrellas, or archive with `absorbed_into`. Bundled built-ins and `skills.disabled` are not offered to it. A skill with `references/`, `templates/`, `scripts/`, or `assets/` must be kept, re-homed with path rewrites, or archived whole — not flattened to another skill's SKILL.md.

`prune_builtins: false` by default. Hub-installed skills (agentskills.io, `.hub/lock.json`) are always off-limits.

"Agent-created" means all of: not in `.bundled_manifest`, not in `.hub/lock.json`, and `.usage.json` has `created_by: agent` or `agent_created: true`. Only the background review fork (`write origin` `background_review`, via `tools/skill_provenance.py`) sets that marker, when it creates an umbrella during its periodic pass (~every 10 agent turns, per the curator page). Foreground `skill_manage(create)`, including `/learn`, is `created_by: learn` and is left alone. Hand-written skills have `created_by: null` and are not curated. `hermes curator adopt` sets the agent marker without resetting `last_activity_at`.

Adoption only governs archival. The background review can patch, rewrite, and add support files to any skill it learns from, including hand-written, hub, bundled, external, and pinned. It may archive only curator-managed, non-pinned skills. `skills.write_approval` gates those writes. A bundled or hub skill edited by the review counts as locally modified, so `hermes update` / `hermes skills update` stop overwriting it.

Before a consolidation pass, a tar.gz snapshot goes to `~/.hermes/skills/.curator_backups/<utc-iso>/skills.tar.gz` (live tree only; `.archive/`, ledger, `.hub/`, and backups are excluded). `hermes curator rollback` restores the newest snapshot and first takes a `pre-rollback` snapshot. Prune-only takes no snapshot; undo is `hermes curator restore`. Snapshots kept: `curator.backup.keep` default 2.

Every skill mutation appends to `~/.hermes/skills/.curator_ledger.jsonl` (actor curator/agent/user; action; before/after sha256 manifests; blobs under `~/.hermes/.curator_backups/blobs/`). Ledger is telemetry, not a gate. Default cap 5 MB (`skills.ledger_max_bytes`). Archive TTL purge is explicit only (`archive_ttl_days` default 0 = never).

## Background review fork (GitHub issues, not official docs)

Issue #94226, open, verified by the reporter on main `f14059f`:

- `memory.nudge_interval` default 10. Incremented once per `run_conversation()` in `agent/turn_context.py`, before the tool loop. Unit is top-level user turns. Resume hydrates `_turns_since_memory = prior_user_turns % interval`.
- Spawn in `agent/turn_finalizer.py` when there is a final response, the turn was not interrupted, `skip_background_review` is false, and (`should_review_memory` or `should_review_skills`).
- A foreground `memory` tool call resets `_turns_since_memory` to 0, which suppresses the scheduled review.
- Skill side: `skills.creation_nudge_interval` is "every N tool-calling iterations" (example config 15 per the issue; issue #42388 says default 10 and "usually unset → effectively on"). `_iters_since_skill` increments per tool iteration in `agent/conversation_loop.py`.
- Mid-run `/steer` does not increment the memory counter. Delegation children have their own counters. Cron and the review fork set `skip_background_review=True`.
- `flush_memories` was removed by merged PR #15696. The periodic nudge is the only automatic built-in memory-extraction trigger. The fork is bounded (`_REVIEW_MAX_ITERATIONS = 16` in `agent/background_review.py`), best-effort, and runs after the response is delivered. It can conclude nothing is worth saving.
- Consequence claimed by the issue: a heavy agentic session of 1–2 user turns with dozens of tool iterations may never fire memory review at the default of 10, while skill review fires because it counts tool iterations. Counters also reset on `/new` (issue #18369). Session-boundary review is a separate open request (#31597). Short sessions can lose durable facts (#58669).

Issue #42388, closed as not planned but labeled `sweeper:implemented-on-main`:

- The fork is spawned if either gate fires, and it inherits the parent's toolset, including the built-in memory tool, regardless of which gate spawned it. Setting `memory.nudge_interval: 0` does not stop skill-triggered forks from writing MEMORY.md / USER.md. The only config-level stop described in the issue was also setting `skills.creation_nudge_interval: 0`.
- The fork sets its own `_memory_nudge_interval = 0` to avoid recursive review, and `skip_memory=True` for external providers, but the built-in memory tool stayed in `valid_tool_names`.
- The requested flag was `memory.background_review_writes`. The closed-as-not-planned label says the behavior is already on current main; this capture did not read the implementing commit, so do not treat the flag name as confirmed.

## Disagreements

- Vectorize (2026-03-31) says session search results are summarized by an LLM call. Official docs say FTS5 returns actual messages with no LLM summarization and no truncation. Prefer the official docs.
- Vectorize says a dedicated memory flush still runs before context compression, and that gateway mode flushes before idle timeout. Issue #94226 says `flush_memories` was removed (PR #15696) and the periodic nudge is the only automatic extraction trigger. Prefer the later GitHub report; treat the flush as historical.
- Vectorize says skills are written reactively after task completion. Official curator docs say foreground creates are user-directed (`created_by: learn`) and only the background review fork marks `created_by: agent`. Both can be true: the user-visible create path and the autonomous path are different.
- Curator page says the background fork creates umbrella skills "~every 10 agent turns". Issue #94226 says skill nudge is per tool-calling iteration (example 15) and memory nudge is per user turn (default 10). "10" is not one unit. Do not collapse them.
- Vectorize's provider table (Hindsight, Honcho, Mem0, OpenViking, Holographic, RetainDB, ByteRover) and LongMemEval scores are vendor marketing. Official docs only name external providers as an optional `memory.provider` (examples: Hindsight, Mem0, Honcho) and say built-in files stay unless explicitly disabled. Do not treat the score table as Hermes documentation.
