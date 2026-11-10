# Cometline Agent Controls

Cometline separates agent behavior controls from the selected model. The August 2026 repository update adds both a planning mode and a reasoning effort setting, carried from the desktop composer through the OpenAPI contract to CometMind and comet-sdk.

## Agent modes

- `auto` is the general execution mode.
- `plan` is explicitly research only. It uses mode specific instructions, advertises research subagents, and has a restricted tool surface.
- Mode selection persists in runtime configuration, so the active session behavior can survive application restarts.

The restriction is important. A plan mode that merely asks the model not to edit still exposes execution tools and invites accidental work. CometMind instead constrains the tools it offers, making the mode boundary enforceable by the runtime.

## Reasoning effort

Reasoning effort is an optional request property. It is supported end to end only where the selected provider accepts it:

1. The composer lets the user choose effort.
2. CometMind validates the provider and exposes the setting in its API types.
3. [[cometline-sdk-internals]] passes the request to compatible provider implementations.

Provider capability validation keeps a UI control from becoming a provider specific failure. The related model catalog preserves protocol metadata, allowing CometMind to route `opencode-go` models to the appropriate protocol implementation.

## Execution safeguards

The runner has increased default step budgets, does not impose a default ACP delegation deadline, and finalizes a response after reaching the work budget. Together these changes favor a clear bounded result over a turn that appears to stall indefinitely.

## Relationships

- [[cometline-agent-runtime]] implements the mode prompt and tool boundaries.
- [[cometline-sdk-internals]] owns normalized provider requests and Responses protocol support.
- [[cometline-frontend]] owns the user controls.
- [[cometline-coding-delegation]] is intentionally constrained in planning mode.

## References

- [[2026-08-03-cometline-current-repo-update.md]]
- [[cometline-agent-runtime]]
- [[cometline-sdk-internals]]
- [[cometline-frontend]]
