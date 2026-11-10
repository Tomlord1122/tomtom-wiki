# Cometline Tool System

The extensible tool registry in `cometmind/internal/tools/` that powers the agent's ability to interact with files, code, memory, jobs, MCP servers, and settings.

## Tool Interface

Every tool implements:

```go
type Tool interface {
    Spec() ToolSpec      // name, description, JSON Schema input
    Execute(ctx, input)  // run the tool, return result
}
```

Registered in `registry.go` and auto-exposed to the agent loop.

## Built-in Tool Categories

### File System Tools

| Tool | File | Purpose |
|---|---|---|
| `read_file` | `readfile.go` | Read text files with line numbers, offset/limit |
| `write_file` | `writefile.go` | Create or overwrite files, parent dirs created automatically |
| `edit_file` | `editfile.go` | Search/replace in existing files, fuzzy matching |
| `list_dir` | `listdir.go` | Non-recursive directory listing |
| `glob` | `glob.go` | Glob pattern matching with `**` support |
| `grep` | `grep.go` | Ripgrep-style content search, gitignore-aware |

### Command Execution

| Tool | File | Purpose |
|---|---|---|
| `run_command` | `runcommand.go` | Shell command with guardrails, timeout, output capture |

### Web Tools

| Tool | File | Purpose |
|---|---|---|
| `web_fetch` | `webfetch.go` | Fetch public URLs, returns readable text |
| `web_search` | `websearch.go` | DuckDuckGo search with optional recency filter |

### Memory Tools

| Tool | File | Purpose |
|---|---|---|
| `create_memory` | `memory_tools.go` | Save a semantic memory (kind, weight, pinned) |
| `search_memories` | `memory_tools.go` | Semantic search over memories |
| `list_memories` | `memory_tools.go` | List active memories with filters |
| `update_memory` | `memory_tools.go` | Modify existing memory |
| `delete_memory` | `memory_tools.go` | Remove a memory |
| `recall_task_outcome` | `recall_task_outcome.go` | Recall recent task outcomes |

### Job Tools

| Tool | File | Purpose |
|---|---|---|
| `propose_job` | `job_tools.go` | Propose work for user confirmation |
| `create_job` | `job_tools.go` | Create immediate todo job |
| `claim_job` | `job_tools.go` | Claim a job for current session |
| `update_job` | `job_tools.go` | Report progress on claimed job |
| `complete_job` | `job_tools.go` | Mark as done |
| `release_job` | `job_tools.go` | Return to todo |
| `list_jobs` | `job_tools.go` | Query job queue |
| `create_scheduled_job` | `scheduled_job_tools.go` | Schedule future/recurring work |
| `list_scheduled_jobs` | `scheduled_job_tools.go` | List schedules |
| `update_scheduled_job` | `scheduled_job_tools.go` | Modify a schedule |
| `delete_scheduled_job` | `scheduled_job_tools.go` | Remove a schedule |

### Skill Tools

| Tool | File | Purpose |
|---|---|---|
| `load_skill` | `skills.go` | Load a skill's SKILL.md into context |
| `write_skill` | `write_skill.go` | Create/update a skill |
| `write_skill_draft` | `write_skill_draft.go` | Create pending skill draft |
| (skill draft management) | `skill_draft_tools.go` | Review, promote, reject drafts |

### Coding Delegation

| Tool | File | Purpose |
|---|---|---|
| `delegate_coding_task` | `delegatecoding.go` | Spawn external coding harness (OpenCode/Claude Code/Codex) |
| `spawn_general_agent` | `spawngeneral.go` | Spawn in-process subagent (research or coding kind) |
| `wait_subagents` | `waitsubagents.go` | Wait for parallel subagent results |

### MCP Tools

| Tool | File | Purpose |
|---|---|---|
| `search_mcp_tools` / `use_mcp_tool` | `mcp/search_tool.go`, `mcp/use_tool.go` | Search the connected catalog (schema on each hit), then call by qualified name. Remote tools are not registered one-for-one. |
| MCP management | `mcp_manage.go` | List servers, test connections, reconnect, OAuth flows |

### Settings & Inbox

| Tool | File | Purpose |
|---|---|---|
| `list_settings` / `get_settings` / `patch_settings` | `settings_tools.go` | Read/write runtime settings |
| `leave_inbox_message` | `inbox_tools.go` | Send notification to user inbox |

### Workspace & Runtime

| Tool | File | Purpose |
|---|---|---|
| `list_mcp_servers` | `mcp_manage.go` | List MCP server connection statuses |
| `reconnect_mcp_server` | `mcp_manage.go` | Force reconnect a server |
| (runtime mount) | `runtime_mount_test.go` | Runtime mount testing |

## Tool Execution Context

`execctx.go` provides sandboxing: workspace-scoped file access, timeout enforcement, and guardrails for command execution. File operations use `fs/` helpers; command execution goes through `sandbox/`.

## Tool Options

`options.go` defines tool registration configuration including provider-safe naming, tool discovery, and runtime enable/disable flags.

## Validation

`validation_test.go` ensures all registered tools have valid JSON Schema input specifications and that argument parsing works correctly.

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
