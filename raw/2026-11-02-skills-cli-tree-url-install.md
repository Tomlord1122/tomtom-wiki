---
title: skills CLI 1.7.4 tree-URL install of one skill
source_url: https://github.com/Tomlord1122/tomtom-skill/tree/master/skills/b2-english-writing
captured_at: 2026-11-02
---

# Observed install, 2026-11-02

Package resolved by `npx`: `skills@1.7.4`.

List (no install):

```bash
npx skills add https://github.com/Tomlord1122/tomtom-skill/tree/master/skills/b2-english-writing --list
```

CLI output:

```text
Source: https://github.com/Tomlord1122/tomtom-skill.git @ master (skills/b2-english-writing)
Found 1 skill
Available Skills
  b2-english-writing
```

Install:

```bash
npx skills add https://github.com/Tomlord1122/tomtom-skill/tree/master/skills/b2-english-writing --skill b2-english-writing -g -y
```

CLI output (abridged to the result lines):

```text
Source: https://github.com/Tomlord1122/tomtom-skill.git @ master (skills/b2-english-writing)
Found 1 skill
Selected 1 skill: b2-english-writing
79 agents
Installing to: Claude Code, Codex, Cursor, Grok Build, OpenCode, Zed
~/.agents/skills/b2-english-writing
  universal: Codex, Cursor, OpenCode, Zed, Amp +17 more
  symlink → Claude Code, Grok Build
Installed 1 skill
Failed to install 1
  b2-english-writing → PromptScript: PromptScript does not support global skill installation
```

On-disk result after that command:

```text
~/.agents/skills/b2-english-writing/SKILL.md
~/.agents/skills/b2-english-writing/scripts
```

Repo layout confirmed via `gh api` before the install: the skill lives at `skills/b2-english-writing/SKILL.md` beside other skills under `skills/`. Latest commit touching that path was `3cae86d` (2026-11-02). Local checkout of the same path was the same commit.
