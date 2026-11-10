---
title: Cometline update — default model roles, transition polish, and shortcut tooltips
source: git repository
source_revision: 099dc20
source_range: 40352ef..099dc20
captured_at: 2026-07-21
---

# Captured source

This note records the Cometline repository changes after the wiki's v0.2.77 snapshot (`40352ef`) through merge commit `099dc20`.

## Commits

- `bf01b4f` — `refactor: transition logic`
- `cb4d131` — `chore: add shimmer effect`
- `0ada671` — `chore(style): add more indicator animation`
- `fc13480` — `fix(cometmind): resolve role LLMs from default model`
- `87de51d` — `refactor(settings): drop activeProviderId for default-only roles`
- `80322d0` — `feat(cometline): add live shortcut tooltips for chrome buttons`
- `099dc20` — merge pull request #62

## Captured change areas

### Default model and helper roles

The settings schema moves from the legacy `activeProviderId` concept to an explicit global pair: `defaultProviderId` + `defaultModelId`. The legacy field remains read-compatible but is stripped on write. Runtime configuration mirrors the pair as `DefaultProviderID` and `DefaultModelID`, while `Provider` and `Model` remain compatibility aliases.

Helper roles (title generation, memory extraction/compaction, autonomy, and skill synthesis) now resolve atomically: a role pin is used only when both provider and model are present; otherwise the global Default pair is used. Provider and model are never mixed across sources. Memory extraction no longer inherits the session's provider/model; it uses its configured pin or Default and skips cleanly when no model exists.

The settings UI drops role pins that refer to disabled or model-less providers, and model lists use only explicitly enabled models. The model-role panel now describes title and memory roles as falling back to Default rather than the current session.

### Conversation transition and rendering

The composer-to-thread first-turn transition was refactored to preserve staged message state across re-renders. Scroll anchoring is disabled during flight and restored afterward. Related changes harden user-message row state, assistant stack placement, mini session sidebar behavior, and terminal/web panel transition integration.

### Thinking indicator

`ThinkingIndicator` now has three visual variants selected by turn phase: `orbit` for thinking/preparation, `eclipse` for tool execution, and `nova` for composing/final response. A sticky variant helper prevents visual flicker when phases change. The indicator gained unit tests and animation styling with reduced-motion handling retained.

### Live shortcut tooltips

A reusable portal-backed `Tooltip` component and shortcut-tooltip utilities were added. Chrome buttons across the app (sidebar, panels, composer, terminal, web panel, and session controls) can show live labels with keyboard shortcuts. Positioning is calculated against the viewport and tested; the portal avoids clipping by app-shell containers.

### Runtime and tests

CometMind configuration, provider factories, memory compaction, settings application, title generation, and server handlers were updated for the new role resolution model. New Go tests cover default adaptation, atomic role resolution, legacy settings behavior, provider selection, and runtime integration. Frontend tests cover tooltip positioning, shortcut labels, thinking variants, transition/scroll behavior, and schema changes.
