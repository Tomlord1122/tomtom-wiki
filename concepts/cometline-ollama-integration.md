# Cometline Ollama Integration

Local LLM support through the user-installed [Ollama](https://ollama.com) daemon for private chat, title generation, and memory embeddings. Cometline does **not** bundle or silently install Ollama.

## Architecture

```
Desktop UI (SvelteKit)
  ├── OllamaProviderPanel  — catalog cards, health check, pull with progress
  ├── Setup Wizard         — onboarding: install Ollama → pull models → assign roles
  └── ollama/client.ts     — renderer-side IPC wrappers
        ↓ IPC (loopback-gated)
Electron main
  ├── ollama-service.cjs   — health (/api/version), list (/api/tags), pull (streamed)
  └── Preload IPC          — cometline:ollama-* channels
        ↓ HTTP (loopback only: 127.0.0.1, localhost, ::1)
Ollama daemon (:11434)
  ├── /api/chat           — OpenAI-compatible /v1/chat/completions (via comet-sdk)
  ├── /api/embeddings     — native Ollama embed API (direct HTTP, not via comet-sdk)
  ├── /api/tags           — list installed models
  └── /api/pull           — pull model with progress streaming
```

## Provider Mapping

Ollama is treated as an OpenAI-family provider in comet-sdk:

- **Provider ID**: `"ollama"` (`config.ProviderOllama`)
- **SDK family**: `config.ProviderOpenAI` — uses OpenAI-compatible `/v1/chat/completions`
- **Chat base URL**: native daemon base + `/v1` (e.g. `http://127.0.0.1:11434/v1`)
- **Auth**: Ollama ignores auth tokens; CometMind sends a dummy key when none is configured
- **Native base URL storage**: stored canonically without trailing `/v1`; `OllamaChatBaseURL()` appends `/v1` when constructing the SDK provider

## Model Catalog

A static, versioned catalog (`OLLAMA_CATALOG_VERSION = 1`) shipped in `cometline/src/lib/ollama/catalog.ts` and duplicated in `electron/ollama-service.cjs`:

| Catalog ID | Pull Name | Size | Role | Architecture |
|---|---|---|---|---|
| `private-memory` | `qwen3-embedding:0.6b` | ~639 MB | embedding | universal |
| `local-companion-efficient` | `gemma4:e2b-mlx` | ~6.5 GB | chat, title | Apple Silicon (MLX) |
| `local-companion-better` | `gemma4:e4b-mlx` | ~8.8 GB | chat, title | Apple Silicon (MLX) |
| `private-memory-lightweight` | `embeddinggemma` | ~622 MB | embedding | universal |

Each entry declares capabilities (`chat`, `title`, `embedding`, `agent`, `extraction`); agent and extraction are not yet advertised (Phase 0).

## Electron Ollama Service

`electron/ollama-service.cjs` provides:

- **Health check**: `GET /api/version` with 4s timeout → `healthy | missing | unreachable`
- **Model listing**: `GET /api/tags` with 10s timeout
- **Diagnostics**: extended health info
- **Model pull**: `POST /api/pull` with NDJSON progress streaming; progress forwarded to renderer via `cometline:ollama-pull-progress` IPC; 1-hour timeout; cancellable via `AbortController`
- **Loopback enforcement**: `assertLoopbackBase()` rejects non-loopback URLs — only `127.0.0.1`, `localhost`, `::1`, `[::1]` are allowed

## Native Ollama Embedder

Memory embeddings use Ollama's native `/api/embeddings` endpoint, **not** the OpenAI-compatible API:

- `memory.NewOllamaEmbedder(baseURL, model)` → `ollamaEmbedder` struct
- Direct HTTP POST per text (sequential, not batched)
- Default model: `qwen3-embedding:0.6b`
- 120s HTTP client timeout
- Resolved in `NewEmbedder()` when provider is `"ollama"`

## Settings Panel

`OllamaProviderPanel.svelte`:
- Health status display with "Check again" button
- Featured catalog cards showing model name, size, description
- "Pull" button per catalog entry with live progress bar
- Advanced custom pull for arbitrary `model:tag` names (validated via `OLLAMA_MODEL_NAME_RE`)
- Auto-merges installed models into provider config after pull
- Opens `ollama.com/download` if Ollama not installed

## Privacy Model

Each role can point at a different provider. Cometline never silently falls back from a local role to a cloud provider. If Ollama is stopped or a model is missing, the role fails with a repairable error.

## Re-embedding Migration

When the embedding model changes (e.g., from OpenAI to Ollama `qwen3-embedding`), a cancellable background re-embed job migrates existing memory vectors. See [[cometline-semantic-memory]] for details.

## References

- [[cometline-agent-runtime]]
- [[cometline-semantic-memory]]
- [[cometline-desktop-shell]]
- [[cometline-provider-sdk]]
