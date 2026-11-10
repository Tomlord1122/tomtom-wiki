# Cometline Inbox & Gateway System

The notification and cross-platform bridge layer: inbox messages for agent-to-user communication, and the Discord gateway for running CometMind as a bot.

## Inbox System

### Purpose

The inbox (`cometmind/internal/inbox/` + `inboxworker/`) is the agent's channel for leaving asynchronous notifications for the user. When the agent completes a job, discovers something, or needs to report results from autonomous execution, it leaves an inbox message.

### Inbox Message (`inbox_messages` table)

| Field | Purpose |
|---|---|
| `title` | Short headline shown in inbox list |
| `body` | Detail text |
| `workspace_id` | Context workspace |
| `job_id` | Link to related job (deep link in UI) |
| `status` | `open` → `archived` |
| `user_reply` | User's response (internalized as memory) |
| `archive_reason` | `replied` or `dismissed` |

### Tool Interface

- `leave_inbox_message` — agent tool to create inbox messages

### Worker (`inboxworker/`)

Background worker that processes inbox messages — internalizing user replies as memories, cleaning up processed messages.

### UI

The desktop shell renders inbox messages as a bell-icon notification list with deep links to related jobs/sessions.

## Discord Gateway (`cometmind/internal/gateway/`)

Runs the same CometMind agent runtime as a Discord bot.

### Configuration

In `~/.cometmind/cometline-settings.json`:
```json
{
  "cometmind": {
    "gateway": {
      "discord": {
        "enabled": true,
        "bot_token_env": "DISCORD_BOT_TOKEN",
        "workspace_path": "/path/to/project",
        "allowed_users": ["user_id"],
        "require_mention": true
      }
    }
  }
}
```

### Features

- **Per-thread sessions** — each Discord thread gets a dedicated CometMind session
- **@mention gating** — bot only responds when mentioned (configurable)
- **Workspace isolation** — Discord sessions are scoped to a configured workspace
- **Skill invocation** — slash commands work via Discord messages
- **Job creation** — jobs created in Discord flow to the same queue

### Session Mapping (`gateway_sessions` table)

Maps Discord identity to CometMind session:
- `platform` = `"discord"`
- `platform_user_id` + `platform_channel_id` + `thread_id` → unique constraint
- `cometmind_session_id` → linked session

### Lifecycle

- Started via `cometmind gateway run --platform discord`
- Managed by the desktop app through `syncDiscordGatewayFromSettings()`
- Gateway restart on settings change (not full sidecar restart)

## Remote companion shape

The Discord gateway is already a remote companion: a remote mouth, a local brain. An iPhone app should be another adapter on the same router, not a second runtime.

| | Discord gateway | iOS companion |
|---|---|---|
| Who thinks | Mac `cometmind` | Same process |
| Tools | Local filesystem, shell, MCP | Same; the phone never executes them |
| Mac asleep | No reply | Read mirrored history; queue a draft |
| Transport | Discord long-lived bot connection into `HandleInbound` | Phone writes a turn; Mac claims it |
| Identity | User and channel allowlist | Separate auth, not the website project |

Claude Code Remote Control matches this shape. OpenCode is a local server plus thin clients on one machine. Claude Desktop is cloud chat with optional local connectors. Do not copy that last shape onto Cometline.

Supabase, if used, is only the mailbox and mirror. `~/.cometmind/cometmind.db` stays the live source of truth. One in-flight run has one owner. Do not mirror full tool arguments or shell output until they are redacted.

The existing Supabase project `tomlord.io` (`cszzflaikiggglzfhaig`) is the personal site: blogs, pages, visits, users. It is not a Cometline database. `public.threads` has RLS disabled; do not enable RLS without policies, and do not add agent tables there.

## Dependencies

- `github.com/bwmarrin/discordgo` — Discord API client
- Gateway lives in CometMind's process, not Electron

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-database]]
- [[cometline-project]]
