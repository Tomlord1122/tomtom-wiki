---
title: Cometline current repository update, agent controls, protocol routing, workspace UX, and setup polish
source_url: file:///Users/tomlord/workspace/personal-project/cometline
source_revision: f7e1ee49907febf4dc74f5b4b9606d03791c5a91
source_range: 099dc20..f7e1ee4
captured_at: 2026-08-03
---

# Captured source

Repository snapshot at `f7e1ee4`, captured from the Cometline monorepo. This update follows the 2026-07-21 snapshot at `099dc20`.

## Agent controls

- Added persisted agent modes, `auto` and `plan`.
- The desktop composer exposes plan controls and propagates the selected mode to CometMind.
- CometMind applies mode specific prompts. Plan mode advertises research only subagents and enforces a restricted tool surface, preventing implementation tools from being offered as if planning could edit the workspace.
- The runner now has larger default step budgets, no default ACP delegation deadline, and a response finalization path after the work budget is reached.

## Reasoning effort and protocol aware model dispatch

- The SDK accepts reasoning effort in requests.
- CometMind exposes it through API and generated frontend types, validates provider support, and documents user facing effort descriptions consistently.
- The desktop model UI provides reasoning effort controls.
- The model catalog retains models.dev protocol metadata. CometMind dispatches `opencode-go` models by protocol.
- comet-sdk has a dedicated API key authenticated OpenAI Responses provider. A shared `responsesproto` package centralizes Responses protocol conversion, streaming, and errors for the new provider and Codex.

## MCP and tool reliability

- MCP OAuth refresh is coordinated so concurrent requests do not race to refresh the same token.
- Tool surface assertions no longer rely on a particular coding harness being installed.

## Workspace and chat UX

- The workspace explorer lazy loads directories and avoids reload loops. Browsing a workspace does not silently change the configured default workspace.
- Electron watches workspace files. External changes refresh workspace views and can open a full page diff.
- Chat adds find in current chat, response snippets, and persisted categorized context chips. File previews can open at selected line ranges.
- Prompt memory retrieval was redesigned and the frontend presents categorized prompt memories.
- The activity group toggle can slide, and the right sidebar gets an explicit icon with improved toggle behavior.

## Desktop polish and setup

- Settings includes a response complete sound control, normalization, and regression coverage.
- The setup wizard and broader UI received polish at the current HEAD.

## Verification evidence

The repository commit history between `099dc20` and `f7e1ee4` includes merged pull requests for memory UX, response bubbles, workspace selection, external file changes, activity group slide, OpenCode Responses support, reasoning effort, enhanced sandbox modes, MCP OAuth refresh coordination, sidebar icon, response complete sound, and setup wizard polish.
