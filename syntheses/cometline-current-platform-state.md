---
title: Cometline Current Platform State
updated: 2026-10-11
sources:
  - README.md
  - cometmind/internal/agent/wiki_review.go
  - cometmind/internal/runtime/skill_curator.go
  - raw/2026-10-08-combined-turn-review.md
  - cometmind/internal/skills/review_write.go
  - cometmind/internal/tools/skills/review_write.go
  - raw/2026-10-08-turn-review-single-flight.md
---

# Cometline Current Platform State

At merge `3206abcb` on 2026-10-08, Cometline is still the local-first desktop agent from the August pages, plus the maintenance loops below. This page replaces the `a446a708` snapshot. The August mechanisms are still present; they are no longer the latest change. PR 194 is merged to `main`; `origin/dev` and `origin/main` both contain `69b134e5`.

## What changed since August

- **One turn-review child, one at a time.** Wiki compile and skill review no longer open two hidden children that read the same transcript. They keep separate counters. Wiki review starts after every tenth completed user turn and clears that counter. Skill review starts after every tenth tool-calling iteration: one assistant step that issues any tool call counts once, and a text-only step does not. A skill start clears the iteration counter and moves the skill transcript window. A wiki-only start does not move that window. `skill_review_started_at`, `wiki_review_started_at`, and `skill_review_last_targets` are no longer read; migration `0044` drops them. The remaining columns are `skill_review_mutating_count` (wiki user turns), `skill_review_tool_iterations`, and `skill_review_count_reset_at`. One child still starts when either gate fires and resets only the gate that fired. `69b134e5` adds a process-wide single flight: while that child runs, a later eligible turn replaces the one pending job instead of starting another child. The running review finishes, then only the latest pending turn runs. Intermediate eligible turns are dropped. The queue does not block the main agent reply. The prompt keeps procedures on `write_skill` and sourced conclusions under `@runtime/wiki/`. That prompt is not the write gate. The review child is registered with `ReviewWriteSkill`, which still presents as `write_skill` but calls `WriteReviewSkill`. Creates are forced to `metadata.cometline.origin: self-improvement`, and an explicit other origin is rejected. Overwrite fails unless the existing skill is already self-improvement. A new skill that overlaps a non-self-improvement skill writes nothing; overlap with another self-improvement skill must be an overwrite. Symlinks are rejected, and this tool cannot promote drafts. Wiki file tools still reject paths outside the wiki. The wiki gate does not require a tool result, so a chat-only window can still open the child; the child may choose not to write. Skill review does not start on that window. Turn review now uses `ExtractionLLM()`, so an empty memory pin falls back to Default, matching memory extraction. See [[cometline-model-roles]]. This page does not claim the old 15-minute wiki cooldown or the old skill-review burst and correction gates still run beside that count.
- **Wiki lookup.** `query-wiki` still reads `@runtime/wiki/index.md` and at most three pages. It does not write. Existing `raw/` files stay immutable.
- **Skill curator.** Separately from the turn-review child, the skill curator keeps a weekly clock: unused self-improvement skills go stale, then archive, then delete after notice. A model merges overlapping skills only when idle; the program checks the merge report before archiving absorbed skills.
- **Desktop and runtime hardening.** The desktop is on SvelteKit 3. Logging moved from `slog` to zap. Cancelled turns stop the agent loop, early subagent results are collected, streamed tokens render immediately, and command trees are killed on timeout or cancel.

## Still true from the August snapshot

- [[cometline-agent-controls]] chooses execution versus research-only planning. Reasoning effort is validated per provider instead of being forwarded blindly.
- [[cometline-sdk-internals]] routes OpenCode Go models by protocol. [[cometline-reasoning-replay]] keeps DeepSeek `reasoning_content` on tool-call history, including an empty string.
- The workspace tree is watched, not scanned once. [[cometline-session-lifecycle]] still owns restoration, disposable-session cleanup, unread output, and composer focus.
- MCP OAuth refresh stays coordinated so concurrent reconnects do not duplicate the refresh.

## What this page is not

It is not a commit log. Three hundred commits landed after `26c9d92`; only the product-level changes above belong here. Concept pages such as [[cometline-agent-runtime]] and [[cometline-semantic-memory]] still hold the stable design.

## References

- [[2026-10-08-combined-turn-review]] — commit `a446a708`, later merged through PR 194
- [[2026-10-08-turn-review-single-flight]] — commit `69b134e5`, merge `3206abcb`
- Workspace `README.md` at the older `de75fe71` snapshot
- `cometmind/internal/agent/wiki_review.go`
- `cometmind/internal/skills/builtin/query-wiki/SKILL.md`
- `cometmind/internal/runtime/skill_curator.go`
- [[2026-08-03-cometline-current-repo-update.md]] — superseded snapshot, kept as raw history
- [[2026-08-09-cometline-session-lifecycle-update.md]] — session lifecycle capture through `26c9d92`
