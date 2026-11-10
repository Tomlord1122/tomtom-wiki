# Cometline API Surface

The HTTP/SSE contract between the desktop shell and agent runtime at `http://127.0.0.1:7700`. Source of truth: `cometmind/openapi.yaml` (OpenAPI 3.1).

## Endpoints by Tag

### Health
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/health` | Server liveness check |

### Models
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/models` | List configured models across enabled providers |

### Workspaces
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/workspaces` | List registered workspaces |
| POST | `/api/v1/workspaces` | Register or resolve workspace by filesystem path |
| DELETE | `/api/v1/workspaces` | Remove unused workspace registration (no sessions) |
| POST | `/api/v1/workspaces/prune-runs` | Remove registrations for deleted directories |

### Files
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/workspaces/files` | List workspace files (gitignore-aware, filterable) |
| GET | `/api/v1/workspaces/files/content` | Read file for preview (text or image data URL, ≤256KB) |

### Wiki
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/wiki/files` | List wiki files |
| GET | `/api/v1/wiki/files/backlinks` | Find backlinks to a wiki file |
| GET | `/api/v1/wiki/files/content` | Read a wiki file |
| PUT | `/api/v1/wiki/files/content` | Write a wiki file |

### Sessions
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/sessions` | List sessions for a workspace |
| POST | `/api/v1/sessions` | Create new session |
| GET | `/api/v1/sessions/{id}` | Get session detail with messages |
| DELETE | `/api/v1/sessions/{id}` | Delete session |
| PATCH | `/api/v1/sessions/{id}` | Update session (title, pin, archive) |
| POST | `/api/v1/sessions/{id}/fork` | Fork session to another workspace |
| POST | `/api/v1/sessions/{id}/messages` | Send message → returns SSE stream |

### Skills
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/skills` | List discovered skills (built-in + workspace + user) |
| POST | `/api/v1/skills/reload` | Reload skill registry from disk |

### MCP
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/mcp/servers` | List configured MCP servers with connection status |
| GET | `/api/v1/mcp/tools` | List MCP-provided tools |
| POST | `/api/v1/mcp/servers/{id}/connection-tests` | Test server connection |
| POST | `/api/v1/mcp/servers/{id}/reconnection-runs` | Force reconnect |
| POST | `/api/v1/mcp/servers/{id}/oauth-flows` | Initiate OAuth flow |

### Memory
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/memory` | List/search memories |
| POST | `/api/v1/memory` | Create memory |
| PATCH | `/api/v1/memory/{id}` | Update memory |
| DELETE | `/api/v1/memory/{id}` | Delete memory |
| POST | `/api/v1/memory/compact` | Trigger manual compaction |
| POST | `/api/v1/memories/reembed-preview` | Preview how many memories need re-embedding for a model change |
| GET | `/api/v1/memories/reembed-jobs` | Get current re-embed job status |
| POST | `/api/v1/memories/reembed-jobs` | Start re-embed migration |
| POST | `/api/v1/memories/reembed-jobs/current/cancellation` | Cancel running re-embed job |

### Storage
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/storage/summary` | Storage usage summary |
| POST | `/api/v1/storage/cleanup` | Clean old logs/sessions |

### Inbox
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/inbox` | List inbox messages |
| GET | `/api/v1/inbox/summary` | Get unread/open inbox summary |
| GET | `/api/v1/inbox/{id}` | Get one inbox message |
| POST | `/api/v1/inbox/{id}/reply` | Reply to inbox message |
| POST | `/api/v1/inbox/{id}/dismiss` | Dismiss inbox message |

## SSE Stream Events

The `POST /api/v1/sessions/{id}/messages` endpoint returns a Server-Sent Events stream. Event types:

| Event | Emitted When |
|---|---|
| `content_block_delta` | Text/tool-call delta from model |
| `reasoning_delta` | Reasoning/thinking block delta |
| `tool_use` | Tool call requested by model |
| `tool_result` | Tool execution completed |
| `turn_complete` | Entire agent turn finished |
| `memory_compaction_completed` | Context compaction ran |
| `subagent_update` | Subagent (spawned/coding) progress |
| `error` | Fatal stream error |

## Code Generation

- **Source**: `cometmind/openapi.yaml`
- **TS client**: `cometline/src/lib/generated/cometmind-api/` via `@hey-api/openapi-ts`
- **Go types**: `cometmind/internal/apigen/types.gen.go` via `oapi-codegen`
- **Regenerate**: `make generate` from repo root

## SSE Event Extension

Adding a new SSE event:
1. Extend `StreamEvent` in `openapi.yaml`
2. Add to `cometmind/internal/event/event.go`
3. Emitter function in same file
4. `make generate`
5. Reducer case in `cometline/src/lib/reducers/chat.ts`
6. Contract test in `cometmind/internal/contract/`

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-architecture]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
