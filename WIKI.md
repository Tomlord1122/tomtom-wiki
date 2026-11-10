# Wiki Schema & Conventions

## Directory Layout

| Path | Purpose |
|------|---------|
| `raw/` | Immutable source captures (never edit after initial save) |
| `entities/` | People, organizations, products, projects |
| `concepts/` | Ideas, frameworks, mental models, terminology |
| `syntheses/` | Cross-topic write-ups connecting multiple entities and concepts |

## File Naming

- **Raw files**: `{YYYY-MM-DD}-{slug}.md` (e.g., `2026-04-02-karpathy-llm-wiki.md`)
- **Entity pages**: `{slug}.md` (e.g., `andrew-karpathy.md`)
- **Concept pages**: `{slug}.md` (e.g., `llm-wiki-pattern.md`)
- **Synthesis pages**: `{slug}.md` (e.g., `ai-tools-landscape.md`)

## Raw File Format

Every raw file must have YAML frontmatter:

```yaml
---
title: Article title
source_url: https://…
captured_at: YYYY-MM-DD
---
```

The body is the captured content, preserved verbatim.

## Page Conventions

- Every page has a `## References` section at the bottom linking back to raw sources
- Cross-references between pages use `[[page-slug]]` wiki-link syntax
- The index.md catalog is updated whenever a new page is created or substantially revised

## Lint Rules

1. Every entity/concept/synthesis page must be listed in `index.md`
2. Every reference in a page's `## References` section must point to an existing raw file
3. Raw files must have valid frontmatter with `title` and `captured_at`
4. Cross-reference links (`[[…]]`) must point to existing pages
5. No orphaned pages (pages not linked from index.md or another page)

## Maintenance

- Run `/llm-wiki lint` periodically to catch drift
- The log.md records every ingest, query, and lint pass with timestamps

## Size budget

The wiki directory must stay under **10GB**. Measure with `du -sh` on `~/.cometmind/wiki` at the start of a Heptabase sync.

Compiled pages are the durable layer. Raw snapshots are evidence, not a second copy of the library.

- Do not append another full card dump when the previous Heptabase snapshot already holds those notes. A later snapshot records topology plus notes that are new or materially changed.
- Fold repeated learning into the existing concept or synthesis page. Do not open a new page for a restatement.
- If the directory reaches 8GB, compact before adding more: drop raw note text already compiled, keep the snapshot frontmatter and the change list, and fix links. Never delete a compiled conclusion that has no replacement.
- If it is still over 10GB after that, stop adding raw text and say so in the inbox note. Do not keep growing past the cap.
