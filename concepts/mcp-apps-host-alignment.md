# MCP Apps host alignment

How a host should line up with Codex and Grok when adding MCP Apps (`io.modelcontextprotocol/ui`), based on local checkouts rather than the spec body.

## What both hosts already share

Both advertise the extension on MCP initialize and name `text/html;profile=mcp-app` in `mimeTypes`. The wider `text/html` appears only in a Codex protocol unit test, not in the Codex MCP client advertisement.

Both treat UI as tool metadata, not as a second tool the model must call. The canonical link is `_meta.ui.resourceUri`. Codex also accepts the flat key `ui/resourceUri` and the Apps SDK key `openai/outputTemplate`.

Neither checkout in [[2026-08-14-codex-grok-mcp-apps-host]] fetches `ui://` HTML or implements the guest `postMessage` bridge. Codex puts `McpAppUi` on the tool-call item and leaves rendering to another process. Grok discovers UI tools and notifies a pager. Advertising the extension is not the same as hosting the view.

## Wire shape to copy

Codex's outward object is `McpAppUi`: `resourceUri` plus `preferredModelDisplayMode` (`inline` or `fullscreen`). The older `mcpAppResourceUri` field is kept with the same value and marked legacy. A new host should emit that shape even if its first renderer only supports one mode. Internal HTML and CSP can ride beside it; they should not rename the public fields.

## Catalog split Grok already has

`_meta.ui.visibility` is an array. Missing means model-visible. Present and lacking `"model"` means the tool stays off the model catalog but remains callable by the app that owns the UI. Grok still lists those tools in a per-server UI index and emits `tools_changed` with `sessionId`, so a background session cannot paint the foreground view.

That split is the extension point for app-only tools (confirmation UI, tools the view calls). Default remains model-visible so ordinary UI tools still appear in search.

## What Cometline now hosts

[[cometline-mcp-integration]] keeps UI metadata, advertises the extension, fetches `ui://` only after the dispatched tool is known, and checks the profile MIME. HTML stays off the SSE frame. `mcp_app_ready` carries Codex-shaped `mcpAppUi` plus an app id. The desktop loads that document into a sandboxed `srcdoc` iframe. `ensureWorkspacePanelVisible()` opens the panel; navigating the web surface to an empty URL clears the view instead.

## What a host still has to build

The open iframe is display-only. Operating the view still needs a guest `postMessage` bridge and a proxy for tools the app calls. Do not infer a double-iframe rule or a method list from the Codex or Grok trees. Do not treat a chart `srcdoc` viewer as the MCP App host.

## References

- [[2026-08-14-codex-grok-mcp-apps-host]]
- [[cometline-mcp-integration]]
