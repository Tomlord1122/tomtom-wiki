# Turn review single flight

Source: parent-turn tool results, 2026-10-08. Local tests of `cometmind/internal/agent` and `internal/runtime` passed after the change. Commit `69b134e5` (`fix(agent): keep one turn review running`) was pushed to `dev`, then included in merge `3206abcb` (`Merge pull request #194 from Cometline/dev`). `gh pr view 194` reported state `MERGED`, mergedAt `2026-10-08T14:00:59Z`, merge commit `3206abcbf48c0b7a609320ed3e6272f55e3772b4`. After fetch, `origin/dev` had no commits ahead of `origin/main`.

## What the commit does

Before `69b134e5`, three eligible reviews started together. `TestTurnReviewCoalescesWhileRunning` failed with three prompts and `running = 0`. After the change, the same test passed: max running is 1, the first prompt contains `first review`, and the second contains `third review` but not `second review`.

The implementation is a process-wide mutex and one pending job. If a review child is running, a later eligible turn replaces that pending job. When the running child finishes, only the latest pending turn starts. Intermediate eligible turns are dropped. The parent turn states that this lock does not block the main agent reply. `go test ./internal/agent/ ./internal/runtime/ -count=1` passed.

## What this source does not establish

This note does not quote the tenth-user-turn counter. It does not say whether the old 15-minute wiki cooldown or the old skill-review burst and correction gates still exist. The weekly skill curator is not part of this diff.
