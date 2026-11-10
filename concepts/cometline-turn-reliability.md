# Cometline Turn Reliability & Provider Streaming

The recent update strengthens the path from provider stream to user-visible turn, especially around Codex responses, incomplete tool calls, idle timeouts, and recovery nudges.

## Incomplete Tool Calls

The agent runner now explicitly handles tool calls that arrive incomplete or malformed rather than assuming every streamed tool call can execute normally. This protects the turn from getting stuck between model output and tool execution.

The behavior is covered by runner and request tests, with corresponding chat reducer coverage for the UI state.

## User Nudges

When a turn cannot continue normally, Cometline gives the user a visible nudge instead of silently ending or leaving the UI in an ambiguous waiting state. The frontend maps the runtime outcome into assistant wait/status presentation.

## Stream Idle Timeout

The stream idle timeout was increased to **10 minutes**, better accommodating long reasoning or slow provider responses while still preventing permanently hung streams.

## Codex Stream Handling

The SDK and Codex adapter were expanded around:

- Response conversion and validation
- Optional output fields
- Streaming event handling
- Reasoning/thinking presentation
- Provider errors
- SSE scanning and incomplete chunks

Dedicated conversion, stream, and validation tests make provider behavior reproducible without live API calls.

## Provider State

Assistant provider/model state is persisted per message. This preserves which provider generated a response when sessions switch models or when multiple provider states are involved in a turn.

## Boundary

The SDK normalizes provider-specific stream behavior; CometMind owns turn recovery and persistence; Svelte reducers own rendering. This keeps reliability fixes distributed across the correct layers rather than hidden in UI workarounds.

## References

- [[2026-07-18-cometline-update-v0.2.76.md]]
- [[cometline-sdk-internals]]
- [[cometline-agent-internals]]
- [[cometline-api]]
- [[cometline-frontend]]
- [[cometline-project]]
