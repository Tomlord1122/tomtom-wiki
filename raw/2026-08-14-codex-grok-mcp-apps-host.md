---
title: Codex and Grok MCP Apps host paths (local checkouts)
source_url: file:///Users/tomlord/workspace/open-source
captured_at: 2026-08-14
---

Notes from reading local clones, not from vendor blogs. Paths are relative to `/Users/tomlord/workspace/open-source`.

## Codex (`codex/codex-rs`)

Extension id: `io.modelcontextprotocol/ui` (`protocol/src/mcp.rs`, `MCP_APP_UI_EXTENSION_ID`).

Initialize advertisement in `codex-mcp/src/rmcp_client.rs` test `mcp_initialize_advertises_client_extensions` uses:

```json
{ "mimeTypes": ["text/html;profile=mcp-app"] }
```

A protocol unit test in `protocol/src/mcp.rs` uses the wider `text/html`. The MCP client path uses the profile MIME.

Resource URI lookup in `core/src/mcp_tool_call.rs` `get_mcp_app_resource_uri` tries, in order:

1. `_meta.ui.resourceUri`
2. `_meta["ui/resourceUri"]` (`MCP_TOOL_UI_RESOURCE_URI_META_KEY`)
3. `_meta["openai/outputTemplate"]` (`MCP_TOOL_OPENAI_OUTPUT_TEMPLATE_META_KEY`)

`protocol/src/items.rs` `McpAppUi` is camelCase `resourceUri` plus `preferredModelDisplayMode` (`inline` | `fullscreen`). `McpToolCallItem.mcp_app_resource_uri` is documented as legacy; prefer `mcp_app_ui.resource_uri`.

Core attaches that metadata to the tool-call item. This checkout does not fetch `ui://` HTML or host a postMessage bridge. `tui/src/inline_visualization/viewer.rs` is a separate srcdoc chart viewer, not the MCP App host.

## Grok (`grok-build/crates`)

`codegen/xai-grok-mcp/src/servers.rs` `make_client_info` inserts `io.modelcontextprotocol/ui` with `mimeTypes: ["text/html;profile=mcp-app"]`.

`into_registration` sets `model_visible` from `_meta.ui.visibility`. If the array is present, the tool is model-visible only when it contains `"model"`. If the field is absent, the default is visible. `register_mcp_tool` in `codegen/xai-grok-shell/src/session/acp_session_impl/mcp_init.rs` skips registration when `model_visible` is false, and still records tools whose meta has `ui.resourceUri` into `ui_tools_by_server`.

`emit_mcp_tools_changed_notifications` sends `x.ai/mcp/tools_changed` with `sessionId` so a background session does not paint the foreground view.

`McpToolEntry` keeps `_meta`. This checkout discovers and notifies; it does not contain an iframe host or postMessage bridge.
