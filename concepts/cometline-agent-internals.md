# Cometline Agent Internals

The core agent execution engine in `cometmind/internal/agent/` — the multi-step loop, context management, and turn lifecycle.

## Runner (`runner.go`)

The central orchestrator that drives the agent loop:

1. **Build request** — Assemble messages, tool definitions, system prompt, retrieved memories, and context summary into a provider request
2. **Stream model** — Send to LLM provider via [[cometline-provider-sdk]], receive streaming events
3. **Persist** — Store user message, assistant response, and tool calls in SQLite
4. **Execute tools** — If model requests tool calls, execute them sequentially in execctx sandbox
5. **Loop** — Feed tool results back to model, continue until natural stop or 50-step limit

### Stream Failure Recovery (`stream_failure.go`)

Handles provider errors mid-stream: retries with backoff, partial result preservation, and graceful degradation when the model connection drops.

## Turn Host (`turn_host.go`)

Manages the lifecycle of a single turn:
- User message ingestion (text + multimodal files)
- Memory injection (relevant memories retrieved before model call)
- Reasoning block accumulation
- Tool call orchestration
- SSE event emission to desktop

## Context Window (`contextwindow.go`)

Manages the limited context window budget:
- Tracks token usage across the conversation
- Triggers compaction when approaching limits
- Prioritizes recent messages + system prompt + retrieved memories
- Handles truncation strategies for oversized conversations

## Compaction (`compaction.go`)

When context window fills up:
1. Summarize older messages into a `context_summary`
2. Store the summary on the session row
3. Mark truncated range with `compacted_until_message_id`
4. Inject summary into future requests as synthetic system context
5. Emit `memory_compaction_completed` SSE event

This allows indefinite conversations without hitting context limits.

## Token Budget (`budget.go`)

Calculates available token budget per turn:
- Model's max context window
- Reserved budget for system prompt + tool definitions
- Reserved budget for output tokens
- Available = max - reserved - current usage

## Coding Policy (`coding_policy.go`)

Determines whether coding-related tool access should be granted:
- Workspace-scoped permissions
- User confirmation gates
- Tool allow/deny lists per session

## Request Assembly (`request.go`)

Builds the provider-agnostic request from:
- Session messages (user, assistant, tool_results)
- Retrieved semantic memories
- Context summary (from compaction)
- Tool definitions (built-in + MCP + skills)
- System prompt (persona SOUL + workspace context)

## Normalization (`normalize.go`)

Sanitizes and validates messages before persistence:
- Strips inlined file content from user messages
- Normalizes tool call argument formats
- Validates role transitions

## Helper-role model resolution

Background model work is resolved through the runtime Default pair rather than the active session model. `Config.ResolveRoleLLM` accepts a role pin only when both provider and model are present; otherwise it returns `DefaultProviderID` + `DefaultModelID` as one atomic pair. This covers title generation, memory extraction/compaction, autonomy, and skill synthesis. A missing usable Default causes memory extraction to skip with a diagnostic instead of guessing.

See [[cometline-model-roles]] for the policy and [[cometline-semantic-memory]] for its memory-specific path.

## Job Progress Hook (`job_progress_hook.go`)

When the agent is executing a claimed job, automatically reports progress milestones back to the job queue via `update_job`.

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-database]]
- [[cometline-tool-system]]
- [[cometline-project]]
