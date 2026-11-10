# Hermes Agent Memory

Nous Research's Hermes Agent keeps future runs cheap by splitting recall into three stores with different budgets.

This is the pattern worth keeping next to [[cometline-semantic-memory]] and [[cometline-web-panel-and-wiki]]: what must shape every turn stays tiny and frozen; reusable procedures live in skills; the transcript stays searchable and is not copied into either.

## Three stores

| Store | Where | When it is in context | What belongs |
|---|---|---|---|
| Durable memory | `MEMORY.md` (2,200 chars) and `USER.md` (1,375 chars) under `~/.hermes/memories/` | Frozen snapshot at session start, always | Stable facts, preferences, corrections, conventions |
| Skills | `~/.hermes/skills/` | Only when the skill is loaded | Procedures: commands, pitfalls, how to verify |
| Session search | `~/.hermes/state.db` (FTS5) | Only when `session_search` is called | Past conversations, scrolled as actual messages |

SOUL.md is personality and operating guidance, not a fourth memory. A document index is also not built-in memory: keep the file and a retrieval procedure outside the character budget.

Profiles isolate memory. Two processes must not share one Hermes home; if they need shared memory, that is an external `memory.provider`, not a second writer on the same files.

## How a write happens

The agent calls a memory tool: `add`, `replace`, or `remove`. There is no read action, because the snapshot is already in the prompt. `replace` matches a unique substring and overwrites the whole entry. Exact duplicates are rejected. Entries are scanned for injection and exfiltration before they are accepted, because they land in the system prompt.

A sentence like "I'll remember that" is not a write. Small models often confirm without calling the tool. `.env` is credentials, not memory. A fact saved mid-session is on disk immediately but invisible to the running session and to any other session that already started. The freeze is deliberate: it keeps the prefix cache stable. Tool responses show the live file; the prompt does not.

Memory does not auto-compact. An add that would exceed the limit returns an error and the current entries. The agent must free space and add in one batch. That is the opposite of silent eviction. When the store is above about 80%, consolidate before adding. Good entries pack related facts. Task logs ("fixed bug X today") do not belong; they expire and steal the budget.

`write_approval: true` stages background and non-CLI writes for `/memory pending`. That is the control for "the agent saved a wrong assumption about me."

## When the review fork runs

Writing is not triggered by which tool was used. After a turn finishes, Hermes may spawn a background review fork that does not touch the live conversation. The fork can decide nothing is worth saving.

Two counters, two units (reporter-verified on main in GitHub issue #94226; not restated as a single number in the official docs):

- Memory review: `memory.nudge_interval`, default 10 **user turns**. One increment per `run_conversation()`, before the tool loop. A foreground memory write resets the counter and suppresses the scheduled review. Mid-run `/steer` and delegated children do not increment it.
- Skill review: `skills.creation_nudge_interval`, counted in **tool-calling iterations** (example config 15; an earlier report says default 10). A single user prompt that drives dozens of tool calls can fire skill review without ever reaching 10 user turns.

The fork is born if either gate fires. Prompt selection is three-way (`_COMBINED_REVIEW_PROMPT`, `_MEMORY_REVIEW_PROMPT`, `_SKILL_REVIEW_PROMPT`), but the tool whitelist is not. It is the memory and skills toolsets together, so the prompt can say "memory only" while `skill_manage(action=create)` is still allowed.

That overlap leaks both ways:

- Skill trigger, memory interval 0: issue #42388. The fork inherited the parent's tools, including built-in memory, so a skill-triggered fork could still write `MEMORY.md` / `USER.md`. Closed as already implemented on main. This page does not confirm the flag name (`memory.background_review_writes` was the request).
- Memory trigger, skill interval 0: issue #82708, open as of 2026-08-09 on main `28e401c30`. `creation_nudge_interval: 0` only skips the skill trigger. A memory-only fork (`review_memory=True, review_skills=False`) still created `amhousing-review-cycle` via `skill_manage`, iterating past validation errors. The memory prompt does not forbid skill writes, and `write_approval` ships off. Suggested fixes (drop `skill_manage` when `review_skills=False`, or strip it whenever the skill interval is 0) were not confirmed as landed.

The review agent is a full forked agent, not a bare completion: daemon thread, `quiet_mode`, at most 16 iterations, inherited credentials and cached system prompt, both nudge intervals forced to 0 so it cannot spawn another review. Dangerous-command approval auto-denies. Writes go through the memory tool and `skill_manage`, not shell. Provenance marks fork-created skills `background_review` / agent-created, distinct from a foreground `/learn` create. Skill policy prefers patching a loaded or umbrella skill, then a support file, and treats a new skill as last resort. Do not record one-off failures or broad tool bans. None of that stops the memory fork from calling the skill tool.

`flush_memories` was removed (PR #15696). The periodic nudge is the remaining automatic extraction trigger for the built-in store. Older writeups that describe a pre-compression flush or an idle-timeout flush are describing a path that later reports say is gone.

The loop only pays off at a session boundary. Messaging chats are one continuous session across restarts. Without `/new`, compaction reruns over a long history, and new memory stays invisible because of the frozen snapshot. CLI invocations are naturally new sessions.

## Skills are a lifecycle, not a note

Foreground skill creates, including `/learn`, are user-directed (`created_by: learn`). Only the background review fork marks a skill `created_by: agent`. The curator manages that second set.

The curator is an inactivity check, not a cron: default every 7 days, and only after 2 hours idle. First run is deferred one full interval. Deterministic pruning is always on: unused 14 days → stale, unused 30 days → archive under `~/.hermes/skills/.archive/`. It does not auto-delete. Pinned skills and skills referenced by any cron job are skipped. Zero uses is not proof a skill is disposable; never-used skills get a grace floor.

LLM consolidation (merge overlapping skills into umbrellas) is off by default. It is expensive — a sweep is described as typically 50–100 aux-model calls — and it rewrites the library. Bundled and hub-installed skills are off-limits to archival unless the user opts in for bundled skills. The review fork may still patch a skill it learns from, including a hand-written or pinned one; archival is the narrower permission. `skills.write_approval` gates those writes. Consolidation snapshots the live tree first; single mutations are also in an append-only ledger.

## What not to copy from secondary writeups

- Official docs: session search returns actual messages, no LLM summarization. A March 2026 vendor article says the opposite. Prefer the docs.
- External providers (Hindsight, Mem0, Honcho, and others) are optional and sit beside the built-in files. Benchmark scores in vendor articles are not Hermes documentation.

## Contrast with Cometline

[[cometline-semantic-memory]] auto-retrieves before a turn and auto-extracts after it into a vector store. Hermes does not embed the durable layer; it spends a fixed character budget in the prompt and refuses the write when full.

Hermes has no growing knowledge wiki. Research that must be cited stays in the original file plus a skill that says how to retrieve it. A short durable fact may enter `MEMORY.md`; a procedure enters a skill; the conversation itself stays in SQLite.

[[cometline-current-platform-state]] now splits the gates the Hermes way: wiki review counts user turns, skill review counts tool-calling iterations, and one child still handles whichever gate fired. The prompt splits procedures onto `write_skill` from sourced conclusions under `@runtime/wiki/`. A weekly curator still archives, then deletes after notice. Hermes archives and does not auto-delete. Its lesson still applies to the combined child: separate spawn gates are not enough if one fork is asked "is anything here worth keeping?" and both write tools stay on the whitelist. The boundary has to be in the question and in the tools that child is allowed to call.

## References

- [[2026-10-08-hermes-agent-memory-skills]]
- [[2026-10-08-hermes-background-review-fork]]
- [[cometline-semantic-memory]]
- [[cometline-web-panel-and-wiki]]
- [[cometline-current-platform-state]]
- [[agentic-reasoning]]
