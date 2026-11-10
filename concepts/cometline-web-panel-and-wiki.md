# Cometline Web Panel, File Tree & Wiki Integration

The v0.2.76 desktop update turns Cometline's web panel from a simple content surface into a workspace-aware browsing and knowledge interface.

## File Tree

`FileTreeBrowser.svelte` presents workspace files inside the panel. It connects to the workspace file API, supports browsing directories, and feeds file selection into preview/editor surfaces.

`FilePreview.svelte` and `FileEditor.svelte` provide in-app inspection and editing. Preview snippets can be inserted into chat context instead of copying whole files manually.

## Mentions

The composer now supports workspace-aware file and directory mentions:

- `@path/to/file` references a workspace-relative file
- `@dir` can reference a directory and its relevant contents
- File mentions are rendered as compact interactive chips
- Path-only references can be understood by the runtime
- Inlined file content can be stripped from display text while preserving the useful reference

This creates a bridge between the visible workspace browser and the agent's file tools.

## Wiki Linking

The runtime now exposes wiki-aware operations and the frontend maintains a cached wiki file index:

- Wiki files are listed from `@runtime/wiki`
- Wiki file backlinks can be queried
- Wiki files can be read and written through the API
- Wikilinks can be recognized and navigated
- Wiki paths are hardened to remain within the runtime wiki boundary

`wiki-file-index.ts` caches up to 500 wiki paths for 30 seconds and de-duplicates concurrent refreshes.

## Markdown Embeds

`markdown/embed.ts` adds compact rendering for:

- Bare HTTP(S) URLs as link chips
- Domain labels and DuckDuckGo favicon icons
- Workspace file mentions
- Runtime wiki mentions
- Slash skill commands

Only HTTP(S) URLs become clickable links. Dynamic HTML values are escaped and routed through the application's external-link handling.

## Panel Context and History

The shell accumulates panel contexts and panel history. A user can inspect a file, browse a wiki page, or open web content while retaining that context as part of the current workspace interaction.

Focus navigation helpers coordinate keyboard movement between the composer, file tree, preview, and web panel.

## API Contract

The OpenAPI surface now includes:

- `listWikiFiles`
- `listWikiFileBacklinks`
- `readWikiFileContent`
- `writeWikiFileContent`

These endpoints are generated into the TypeScript client and backed by CometMind server handlers.

## Why It Matters

Cometline is no longer only a chat UI with file attachment. The workspace, the persistent wiki, and the agent conversation are becoming one navigable context surface: browse → preview → mention → reason → update.

## References

- [[2026-07-18-cometline-update-v0.2.76.md]]
- [[cometline-frontend]]
- [[cometline-api]]
- [[cometline-project]]
