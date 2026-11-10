---
title: Hermes background review fork — shared fork, separate prompts
source_url: https://github.com/NousResearch/hermes-agent/issues/82708
captured_at: 2026-10-08
---

Compiled from public pages fetched this turn. Not a verbatim dump. Secondary writeups describe source layout; they are not the Hermes repository. Where they disagree with the open GitHub issue, prefer the issue.

## Sources

- GitHub issue #82708 (open, labeled type/bug, tool/memory, tool/skills), opened 2026-08-09 by fallleaves, reproduced on main `28e401c30` (2026-08-09): https://github.com/NousResearch/hermes-agent/issues/82708
- Jamie Zhang, "The Self-Improvement Agent in Hermes: A Deep Dive", 2026-05-29 (secondary; page truncated mid-section 14): https://akjamie.github.io/post/2026-05-29-self-improvement-agent-deep-dive/
- James' Growth Diary via BestHub, "What the Agent Does While Idle", 2026-05-30 (secondary distillation; original source behind a login redirect, not fetched): https://www.besthub.dev/articles/what-the-agent-does-while-idle-asynchronous-background-review-after-a-conversation-8dc83f6cac4a

Not fetched this turn: GitHub issues #97471 and #105921 (search hits only), and https://gyliu513.github.io/jekyll/update/2026/04/20/hermes-agent.html.

## What the open issue shows

`skills.creation_nudge_interval: 0` is documented as the switch that stops background review from creating skills. It only disables the skill-review trigger. The memory review fork, still fired by `memory.nudge_interval` (default 10), keeps `skill_manage` on its tool whitelist, and `_MEMORY_REVIEW_PROMPT` does not forbid skill writes.

Observed 2026-08-08, Telegram gateway session `20260808_134933_8501be48`, with skill nudge 0 and memory nudge 10:

- 18:49:50 the fork's user message began "Review the conversation above and consider saving to memory if appropriate."
- 18:51:17 `skill_manage create` failed (description 205 chars).
- 18:51:54 `skill_manage create` failed (YAML frontmatter parse error).
- 18:52:05 `skill_manage create` succeeded. New skill: `amhousing-review-cycle`. File mtime matched.

The skill-review trigger did not fire. The memory fork created the skill anyway, iterating past validation errors. `write_approval` ships off by default, so the write was unapproved.

Root cause as stated in the issue, not re-verified against the tree:

- `agent/turn_finalizer.py`: `_should_review_skills` requires `_skill_nudge_interval > 0`. `_should_review_memory` only requires `_memory_nudge_interval > 0`. Spawn was `review_memory=True, review_skills=False`.
- `agent/background_review.py::spawn_background_review_thread` then selects `_MEMORY_REVIEW_PROMPT`: save to memory if appropriate, otherwise say "Nothing to save." No skill instructions, and no prohibition on `skill_manage`.
- The fork whitelist is "memory and skill management tools; everything else is denied", so `skill_manage(action=create)` is allowed on a memory-only review.

Suggested fixes in the issue, not confirmed as landed:

- Option A: when `review_skills=False`, drop `skill_manage` (and optionally `skills_list` / `skill_view`) from the whitelist, or hard-line the memory prompt: do not create, patch, or delete skills.
- Option B: honor `creation_nudge_interval: 0` at the fork itself, stripping `skill_manage` no matter which trigger fired.

Related issues named in #82708 but not fetched: #66350 (no relevance gate before patch/write), #70128 (approval gate ships off), #64926 (skills not read-only by config), #67582 (near-duplicate blocking), #26298 and #21315 (confirmation gates), #41465 (whitelist blocked `read_file`), #15204 (non-skill side effects after creating a skill).

## How the fork is described elsewhere

Both secondary writeups agree on the shape, with file-path drift (`run_agent.py` vs `agent/background_review.py`, `agent/agent_init.py`, `agent/conversation_loop.py`). Treat paths as the author's map, not a commit pin.

- Two independent counters. Memory: `memory.nudge_interval`, default 10, counted in user turns. Skills: `skills.creation_nudge_interval`, default 10, counted in tool-calling iterations, and only if `skill_manage` is in `valid_tool_names`.
- After a successful, non-interrupted final response, if either flag is set, one daemon thread runs one review agent on a snapshot of the completed transcript. Interrupted turns do not review. The user does not wait. Process exit abandons the thread. Review is best-effort, not transactional.
- Prompt selection is three-way: both flags → `_COMBINED_REVIEW_PROMPT`; memory only → `_MEMORY_REVIEW_PROMPT`; else `_SKILL_REVIEW_PROMPT`. The memory prompt is about who the user is. The skill prompt is aggressive ("a pass that does nothing is a missed learning opportunity").
- The review agent is a full `AIAgent` (`quiet_mode=True`, `max_iterations=16`), not a bare completion. It inherits credentials, model, base URL, session id, and the cached system prompt (PR #17276 claimed ~26% end-to-end cost cut; not re-measured here). It sets both nudge intervals to 0 so it does not spawn another review. Dangerous-command approval auto-denies so the daemon thread cannot block on stdin. `skip_memory=True` skips external memory plugins.
- Tool boundary is a thread-local whitelist from `get_tool_definitions(enabled_toolsets=["memory", "skills"])`. Shell, web, browser, project edits, and messaging are denied. That whitelist is per toolset, not per review purpose — the same fact #82708 reports as the kill-switch bypass.
- Skill writes go through `skill_manage`: create, edit, patch, delete, write_file, remove_file. Layout is `~/.hermes/skills/<name>/SKILL.md` plus `references/`, `templates/`, `scripts/`, `assets/`. Priority is patch a loaded skill, then an umbrella, then add a support file, then create a new class-level umbrella. Do not save one-off incidents, missing binaries, missing credentials, fresh-install errors, transient failures, or broad bans ("browser tools do not work").
- Provenance: the fork sets `_memory_write_origin = "background_review"`. On create, `is_background_review()` marks the skill agent-created so the curator can treat it differently from a foreground / user-directed create. Foreground `SKILLS_GUIDANCE` still tells the main agent to save after a complex task (5+ tool calls) or to patch a skill that proved wrong; that path competes with the user's task, which is why the background pass exists.
- After the fork, `summarize_background_review_actions` scans new tool results and may print a one-line "Self-improvement review" summary. Preferences recorded only in `MEMORY.md` can be missed later if the matching skill is not patched too.

## What this adds to the 2026-10-08 capture

The earlier raw note ([[2026-10-08-hermes-agent-memory-skills]]) already had the inverse leak from #42388: a skill-triggered fork could still write `MEMORY.md` when `memory.nudge_interval` was 0, because the fork inherited the parent's tools. #82708 is the other direction, still open as of this fetch: a memory-only fork can still create skills when `creation_nudge_interval` is 0, because the whitelist is memory-and-skills even when the prompt is memory-only.

Separate counters and separate prompts do not separate the write. The stores are separate (`USER.md` / `MEMORY.md` vs skills vs session DB). The judgment is one fork that can call both tools.
