---
title: vercel-labs/skills CLI — discovery depth, --skill, and source paths
source_url: https://raw.githubusercontent.com/vercel-labs/skills/main/README.md
captured_at: 2026-08-14
---

# Capture note

Fetched 2026-08-14 for the question of whether a nested directory lets `npx skills add` install one skill family only. This file is an excerpt, not the full README. Agent tables and the tail of the discovery list were truncated by the fetch tool. Do not treat this file as a complete copy of upstream.

## Sources

- README (raw): https://raw.githubusercontent.com/vercel-labs/skills/main/README.md
- Issue #1506 (closed by #1866 on 2026-08-04): https://github.com/vercel-labs/skills/issues/1506
- Issue #413 (still Open on fetch): https://github.com/vercel-labs/skills/issues/413

## README excerpts (main, fetched 2026-08-14)

`--skill` installs by skill name, not by directory:

```bash
npx skills add vercel-labs/agent-skills --skill frontend-design --skill skill-creator
npx skills add vercel-labs/agent-skills --skill '*' -a claude-code
```

Source formats include a GitHub tree URL that points at one path, a local directory, and a bare git URL that does not encode a subdirectory:

```bash
npx skills add https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines
npx skills add ./my-local-skills
npx skills add git@github.com:vercel-labs/agent-skills.git
```

Azure DevOps has an explicit path query (`path=/skills/web-design`). The README does not document a `--path` flag for `skills add`.

Skill discovery, as documented on `main`:

> The CLI searches for skills in these locations within a repository. Each skill container directory is walked up to three levels deep, covering flat layouts (`skills/<name>/SKILL.md`) and catalog layouts with one or two category levels (`skills/<category>/<name>/SKILL.md` or `skills/<category>/<category>/<name>/SKILL.md`). A `SKILL.md` discovered at a shallower level shadows anything nested below it. Use `--full-depth` to also discover `SKILL.md` files outside these container directories (e.g. under `examples/` or `tests/`).

If no skills are found in the standard locations, a recursive search is performed. Plugin-manifest skill paths are searched at their declared depth and are not subject to the depth-3 catalog walk.

`metadata.internal: true` hides a skill from normal discovery unless `INSTALL_INTERNAL_SKILLS=1`.

## Issue #1506 (closed)

Opened 2026-06-23 against `skills@1.5.13`. Claim at that time: `npx skills add google/skills` discovered root and single-nested skills only, and ignored double-nested paths such as `skills/ads/google-mobile-ads/google-mobile-ads-get-started`. The reporter knew `--full-depth` found them, but that flag is not a family filter. Closed as completed in PR #1866 on 2026-08-04.

The 2026-08-14 README text above is the later documented behavior: container directories are walked three levels, which covers one or two category folders under `skills/`.

## Issue #413 (open on 2026-08-14)

Local paths can be a group root (`npx skills add ./go-skills/ --list` discovers `go-skills/skills/`). GitHub HTTPS tree URLs can name a subdirectory. SSH and non-GitHub git URLs resolve from the repo root and have no documented `--path`. `--full-depth` finds every skill in the repo and does not filter to one group. The requested `--path` / `--subdir` flag was not in the README options table on this fetch.
