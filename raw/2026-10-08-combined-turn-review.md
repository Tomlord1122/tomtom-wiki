# Combined turn review

Source: parent-turn tool results, 2026-10-08. Local tests of `cometmind/internal/agent`, `internal/tools`, and `internal/runtime` passed. Commit `a446a708` (`feat(agent): review skills and wiki together`) is on branch `feat/combined-turn-review`, PR https://github.com/Cometline/cometline/pull/193, base `dev`. Diff stat from the same turn: 10 files, +227/−323.

## What the commit claims

Before this commit, wiki compile and skill review each opened a hidden child after a turn. Wiki started from a successful tool result. Skill review started from a burst of edits, an accumulated count, or a correction. The two children read the same transcript and could save the same takeaway twice.

`a446a708` replaces those two children with one turn-review child that handles both jobs. It starts after every tenth user turn and resets that count when it starts. The prompt keeps procedures on `write_skill` and sourced conclusions under `@runtime/wiki/`. Wiki file tools still reject paths outside the wiki.

## Files in the commit

- `cometmind/internal/agent/post_turn.go`
- `cometmind/internal/agent/runner.go`
- `cometmind/internal/agent/skill_review.go`
- `cometmind/internal/agent/skill_review_policy.go`
- `cometmind/internal/agent/skill_review_policy_test.go`
- `cometmind/internal/agent/skill_review_test.go`
- `cometmind/internal/agent/wiki_review.go`
- `cometmind/internal/agent/wiki_review_test.go`
- `cometmind/internal/runtime/runner.go`
- `cometmind/internal/tools/registry.go`

## What this source does not establish

The tool output does not quote the new trigger code. It does not say whether the 15-minute wiki cooldown, the web-fetch gate, or the skill-review burst/correction gates still exist beside the tenth-user-turn gate. The weekly skill curator is not part of this diff. PR 193 was open when this note was written; it was not merged.
