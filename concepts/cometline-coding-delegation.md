# Cometline Coding Delegation (ACP)

The Agent Coding Protocol subsystem (`cometmind/internal/acp/`) that delegates complex coding tasks to external CLI-based coding agents.

## Architecture

```
CometMind Agent
  → delegate_coding_task tool
    → acp/manager.go — selects and spawns harness
      → acp/cli.go — CLI profile for target harness
        → Child process (OpenCode / Claude Code / Codex)
      → acp/runner.go — streams progress back
    → Results returned to parent session
```

## Supported Harnesses

| Harness | CLI Profile | Auth |
|---|---|---|
| **OpenCode** | `opencode` | Local |
| **Claude Code** | `claude` | API key |
| **Codex** | `codex` | Subscription (`~/.codex/auth.json`) |

## Key Design Decisions

### Fixed, Non-Interactive Profiles

The CLI profiles in `cli.go` are pre-configured and not user-editable. Users pick which harness to use; the command, arguments, and permission behavior are owned by CometMind. This prevents accidental misconfiguration and ensures consistent behavior.

### Session Tracking

Delegated coding tasks create child sessions linked to the parent via `parent_session_id` with `delegation_status` tracking the lifecycle:

| Status | Meaning |
|---|---|
| `pending` | Task queued, waiting to start |
| `running` | Harness executing |
| `awaiting_user` | Harness needs user input |
| `awaiting_permission` | Permission gate triggered |
| `completed` | Task finished successfully |
| `failed` | Task errored |
| `cancelled` | User cancelled |

### Progress Streaming

The runner (`runner.go`) captures stdout/stderr from the child process and streams progress back to the parent session as SSE events. Results include:
- `output_summary` — high-level summary
- `acp_session_id` — harness-specific session ID for debugging
- Token usage tracking

### Subagent Kinds

The `subagent_kind` field distinguishes:
- `general` — in-process subagent spawned via `spawn_general_agent`
- `acp` — external coding harness via `delegate_coding_task`

### In-Process Subagents (`spawngeneral.go`)

In addition to external harnesses, CometMind can spawn lightweight in-process subagents:
- **kind=research** — read-only exploration (default)
- **kind=coding** — may edit files and run commands
- Results collected via `wait_subagents` tool
- Runs within the same CometMind process, no external binary

## Manager (`manager.go`)

The `acp.Manager` handles:
- Harness selection based on user preference (`default_harness` in settings)
- Child process lifecycle (spawn, monitor, kill on timeout)
- Session creation and status tracking
- Cleanup of orphaned child processes

## Configuration

```json
{
  "acp": {
    "default_harness": "opencode"
  }
}
```

Stored in `~/.cometmind/cometline-settings.json` under `cometmind.acp`.

## Testing

- `cli_test.go` — CLI profile construction
- `delegatecoding_test.go` + `delegatecoding_cancel_test.go` — tool-level integration tests

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-agent-internals]]
- [[cometline-tool-system]]
- [[cometline-project]]
