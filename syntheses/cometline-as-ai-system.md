# Cometline as an AI Engineering System

A cross-topic synthesis connecting Cometline's architecture to broader AI engineering principles covered in the wiki.

## The System Surrounds the Model

[[2026-ai-engineering-systems]] posits that AI capability depends on the surrounding system, not just the model. Cometline embodies this principle across every layer:

### Provider Agnosticism (comet-sdk)

The [[cometline-provider-sdk]] isolates model-specific details behind a single `cometsdk.Provider` interface. This means the agent runtime never knows or cares which model is answering — it just receives streaming events. This mirrors [[ai-gateways]] principles of provider-agnostic routing.

### Agent Loop as System Design

The [[cometline-agent-runtime]]'s multi-step loop (model call → persist → execute tools → continue) is a systems-level decision, not a model capability. The 50-step cap, tool execution ordering, and SSE streaming are all infrastructure choices that shape what the model can accomplish. This aligns with [[agentic-reasoning]] concepts of planning, tool use, and feedback loops.

### Memory as Infrastructure

[[cometline-semantic-memory]] treats memory as persistent infrastructure rather than a prompt-engineering trick. Auto-retrieval, extraction, and compaction are system-level features that make the model appear to "remember" across sessions — exactly the kind of surrounding system that [[ai-assisted-software-development]] identifies as critical to AI-augmented workflows.

### Desktop as Native Experience

The [[cometline-desktop-shell]]'s Electron + SvelteKit architecture reflects the [[native-feel-cross-platform-desktop]] tradeoff: choosing WebView-based tech while investing in native-feel details (tray icon, keyboard shortcuts, mini-window) to avoid feeling like a wrapped website.

### Jobs and Autonomy

[[cometline-jobs-scheduling]] extends the system beyond synchronous chat. The durable work queue, cron scheduling, and autonomous job worker create a system where work can be queued, scheduled, and executed asynchronously — making the agent a persistent worker rather than a fire-and-forget chatbot.

### MCP as Extensibility

[[cometline-mcp-integration]] follows the Model Context Protocol to connect external tools, reflecting the [[cloud-native-ai-infrastructure]] principle of composable infrastructure. The full OAuth flow implementation (RFC 9728, 8414, 7591) shows the system investing in secure, standard protocols rather than ad-hoc integrations.

## The One-Way Dependency Rule

Cometline's strict `cometline → cometmind → comet-sdk` dependency direction is a systems design choice that prevents circular coupling. Each layer can be tested and evolved independently, and the contracts between them (OpenAPI spec, Provider interface, SSE events) are load-bearing.

## Self-Improvement Loop

The closed-loop self-improvement capability — where Cometline can modify its own codebase using coding-harness delegation — creates a meta-system where the agent runtime can improve the desktop shell that hosts it. This is a rare architectural property: the system can reshape its own container.

## References

- [[cometline-project]]
- [[cometline-architecture]]
- [[2026-ai-engineering-systems]]
- [[2026-07-16-cometline-agents.md]]
