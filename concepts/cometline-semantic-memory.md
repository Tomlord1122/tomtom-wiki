# Cometline Semantic Memory

Cometline's persistent memory layer that automatically learns and retrieves context across sessions, keeping the companion aware of user preferences, decisions, and project details.

## How It Works

### Auto-Retrieve

Before each agent turn, relevant memories are retrieved from the SQLite-backed vector store and injected into the conversation context. This means the companion "remembers" past discussions and preferences without the user re-stating them.

### Auto-Extract

After conversations, new facts, preferences, and project details are extracted and stored as durable semantic memories. The extraction runs asynchronously after each turn.

### Workspace-Scoped

Memories are tied to the project/workspace they were created in. Switching workspaces changes the memory pool.

### Compaction

Over time, related memories can be compacted — merged, summarized, or deduplicated — to keep the memory store efficient. Manual compaction can be triggered; automatic compaction may also run. The `memory_compaction_completed` SSE event reports results.

## Memory Kinds

- **preference** — user preferences (e.g., "prefers dark mode", "uses Go for backend")
- **fact** — factual knowledge (e.g., "project uses SQLite", "API runs on port 7700")
- **project** — project-specific details (e.g., "Cometline is a monorepo")
- **task_outcome** — results of completed tasks for future reference

## Management

Memories can be browsed, searched, edited, and deleted through:
- Settings UI in the desktop shell
- Built-in memory tools available to the agent

## Technical Implementation

- Stored in `~/.cometmind/cometmind.db` (SQLite)
- Uses embedding-based semantic search for retrieval
- Weighted by importance (`base_weight`), with optional pinning to prevent decay
- Inbox messages from the agent can be internalized as memories

## Embedding Providers

Memory embeddings support multiple backends via `NewEmbedder()`:

| Provider | Implementation | API |
|---|---|---|
| `openai` / `openai-compatible` / `opencode-go` | `openaiEmbedder` | `/v1/embeddings` (OpenAI-compatible) |
| `ollama` | `ollamaEmbedder` | `/api/embeddings` (native Ollama, direct HTTP) |

The Ollama embedder sends sequential (non-batched) requests per text with a 120s timeout. Default model: `qwen3-embedding:0.6b`.

## Re-embedding Migration

When the embedding model changes (e.g., switching from OpenAI to Ollama `qwen3-embedding`), existing memory vectors may be incompatible. A cancellable background re-embed job (`memory_reembed_jobs` table) handles the migration:

- **Preview**: `POST /api/v1/memories/reembed-preview` estimates how many active memories need migration
- **Start**: `POST /api/v1/memories/reembed-jobs` begins background migration with configurable `force` flag
- **Current job**: `GET /api/v1/memories/reembed-jobs` returns the latest job (pending/running/completed/failed/cancelled)
- **Cancel**: `POST /api/v1/memories/reembed-jobs/current/cancellation` stops an in-progress job

The job processes memories in cursor-based batches, re-embedding each with the new model. Old vectors remain in the retrieval index until the new ones are complete, so memory search keeps working during migration.

## Contrast

[[hermes-agent-memory]] does not embed its durable layer. It injects two char-capped files as a frozen prompt snapshot and refuses a write that would overflow, rather than compacting silently. Session transcripts stay in SQLite and are searched only on demand. Cometline's auto-extract-after-every-turn is a different bet: more capture, a vector index, and compaction instead of a hard character ceiling.

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
- [[hermes-agent-memory]]
