# Cometline Terminal Panel

In-app terminal panel powered by xterm.js + Electron PTY, scoped per chat session and accessible as a workspace surface alongside the web panel.

## Architecture

```
SvelteKit renderer
  ├── TerminalPanel.svelte      — panel shell: toolbar, start/stop, surface switcher
  ├── TerminalInstance.svelte   — xterm.js Terminal + FitAddon, selection context
  └── terminal.svelte.ts        — reactive store: snapshots, listeners, IPC bridge
        ↓ IPC
Electron main (main.cjs)
  ├── createTerminal()          — node-pty spawn with shell
  ├── terminalSessions Map      — per-session pty + metadata
  ├── write/resize/terminate    — I/O and lifecycle
  └── IPC events                — cometline:terminal-data, cometline:terminal-exit
```

## Lifecycle

1. **Start**: `terminalStore.start(sessionId, workspacePath)` → IPC `createTerminal` → Electron spawns `node-pty` with default shell (zsh/bash) in workspace directory
2. **I/O**: Keystrokes → `terminalStore.write()` → IPC → PTY stdin; PTY stdout → IPC `cometline:terminal-data` → store dispatches to xterm.js and subscribers
3. **Resize**: `ResizeObserver` on terminal container → `FitAddon.fit()` → `terminalStore.resize()` → IPC → PTY resize
4. **Terminate**: Stop button → confirm modal → `terminalStore.terminate()` → IPC → PTY kill
5. **Exit**: PTY process exits → IPC `cometline:terminal-exit` → store cleans up, closes panel for that session

## Configuration

| Setting | Value |
|---|---|
| Terminal engine | `@xterm/xterm` + `@xterm/addon-fit` |
| PTY backend | `node-pty` (Electron main) |
| Scrollback | 10,000 lines |
| Output cap | 2,000,000 characters (rolling window) |
| Font | `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas` at 12px |
| Theme | Dark: `#171717` background, `#f4f4f5` foreground |

## Workspace Surface Integration

The terminal panel is one of the workspace panel surfaces (alongside the web panel), toggled via:

- **Cmd+J**: Switch to terminal panel
- **Cmd+O**: Switch to web panel
- Surface switcher buttons in the panel toolbar

The `shellStore` manages which panel is open (`terminalPanelOpen`) and which pane has focus (`focusedPane`).

## Terminal Selection as Context

Selected text in the terminal can be added to the chat context:

- Mouse-up triggers selection extraction (up to 50,000 characters)
- Floating "Add to context" button appears next to selection
- Context is stored with `kind: 'terminal'` and `source: 'terminal://<sessionId>'`
- Composer focus is requested after adding

## CometMind Terminal Context

When terminal context is sent in a chat message, it carries `source: 'terminal://<sessionId>'`. Server-side validation in `messages.go` requires this exact scheme format.

## Store Design

`terminalStore` (in `stores/terminal.svelte.ts`):
- `sessionIds` — active terminal sessions
- `getSnapshot(sessionId)` — returns `TerminalSnapshot` (status, output, workspace)
- `start/write/resize/terminate/remove` — IPC-backed operations
- `subscribe(sessionId, listener)` — push-based data streaming for xterm.js
- Initializes by listing existing terminals from Electron on first call

## Shell Selection

The default shell is detected per-platform in Electron main:
- macOS/Linux: `SHELL` env var → `bash` fallback
- Windows: `powershell.exe` or `cmd.exe`

## References

- [[cometline-desktop-shell]]
- [[cometline-frontend]]
- [[cometline-agent-runtime]]
