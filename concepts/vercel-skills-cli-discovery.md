# Vercel skills CLI discovery

`npx skills add` (package `skills`, repo `vercel-labs/skills`) installs agent skills by discovering `SKILL.md` files. A nested folder name is not a family selector.

## What selects a subset

- `-s, --skill <names...>` filters by the skill `name` in frontmatter. `'*'` means all discovered skills. Repeating `--skill` installs several named skills. It does not match a parent directory.
- A GitHub tree URL such as `https://github.com/<owner>/<repo>/tree/<branch>/<dir>` makes that directory the search root. Pointing it at one skill directory installs only skills discovered under that root. On 2026-11-02, `skills@1.7.4` accepted `tree/master/skills/b2-english-writing` for a skill that already lives beside others under `skills/`. `--list` reported `Found 1 skill`. The branch in the URL must be the branch that contains the directory (`master` here, not `main`).
- A local path works the same way: `npx skills add ./family/` searches that directory, not the repo above it.
- Azure DevOps URLs can pass `path=` and `version=`.
- There is no documented `--path` for generic git or SSH URLs. `git@host:org/repo.git` is searched from the clone root. Issue #413 was still open on 2026-08-14 asking for that flag.

## How deep discovery goes

On `main` as of 2026-08-14, each skill container (`skills/`, `.agents/skills/`, `.claude/skills/`, and the other listed agent dirs) is walked **three levels**:

| Layout | Found by default |
| --- | --- |
| `skills/<name>/SKILL.md` | yes |
| `skills/<category>/<name>/SKILL.md` | yes |
| `skills/<category>/<category>/<name>/SKILL.md` | yes |
| deeper than that, or outside those containers | only with `--full-depth`, a plugin manifest path, or the fallback recursive search when nothing was found in standard locations |

A `SKILL.md` at a shallower level shadows anything nested under it. `--full-depth` is not a family filter: it also picks up skills under `examples/` and `tests/`.

This three-level walk is newer than issue #1506 (opened 2026-06-23 against `skills@1.5.13`, when double-nested skills were missed). That issue was closed by PR #1866 on 2026-08-04. Older CLIs may still miss `skills/<family>/<name>/SKILL.md`.

## Family layout

Putting one family in `skills/<family>/<skill>/SKILL.md` does **not** make `npx skills add owner/repo` install only that family. Current CLI discovers those skills along with every other skill under `skills/`.

To install one family with one path:

```bash
npx skills add https://github.com/<owner>/<repo>/tree/main/<family>
```

The tree URL is the search root, so it does not need a nested `skills/` container of its own. On 2026-11-02 the URL ended at `skills/b2-english-writing`, whose only skill file is `SKILL.md` plus `scripts/`. `--list` found that one skill and did not list the sibling directories under `skills/`. A family buried as `skills/<family>/...` is still not a separate search root when the source is the repo URL.

For a flat `skills/t-*/SKILL.md` layout, name the skills with `--skill`. An extra nesting level does not buy a family install, and on a pre-#1866 CLI it can hide them.

`metadata.internal: true` hides a skill unless `INSTALL_INTERNAL_SKILLS=1`. That hides from discovery; it does not group a family.

## Global install side effect

`npx skills add <tree-url> --skill <name> -g -y` with `skills@1.7.4` wrote the skill to `~/.agents/skills/<name>/`, including files beside `SKILL.md` (`scripts/` in this case). The same run reported `Failed to install 1` for PromptScript: "PromptScript does not support global skill installation." That line did not remove the `~/.agents/skills` copy. Cometline discovers `~/.agents/skills`, so a separate copy under `~/.cometmind/skills` was not required.

## References

- [[2026-08-14-vercel-skills-cli-discovery]]
- [[2026-11-02-skills-cli-tree-url-install]]
