# Cometline MCP Integration

CometMind's Model Context Protocol client that connects to external MCP servers, exposing their tools to the main agent with full OAuth support.

## Architecture

CometMind acts as an MCP client. Connected remote tools are **not** registered one-for-one on the native tool list. The main agent gets two fixed tools instead: `search_mcp_tools` and `use_mcp_tool`. Built-in tools stay native. `list_mcp_servers` and `reconnect_mcp_server` stay.

`search_mcp_tools` tries an exact name match first, then keyword-ranks over server, tool name, description, and parameter names. Each hit includes the full input schema. Default limit is 5, max 8. A qualified name (`mcp_{serverId}_{toolName}`) returns that match. A bare tool name returns every server that exposes it, so two servers both named `query` are not collapsed to whichever catalog map walk happened first. `use_mcp_tool` dispatches the live call. A bare name is accepted only when exactly one catalog entry has that tool name; an ambiguous bare name fails and the caller must pass the qualified name from search. Tools hidden by `allowedTools` are neither searchable nor callable.

An optional user-authored server description flows from settings JSON through config onto each catalog entry and into a short connected-server reminder (id, tool count, optional one-line description). Empty description keeps the old line. Unchanged catalogs are not resent: the reminder fingerprint is cleared only when compaction actually changes the session, not on every successful compact check. A no-op check must leave the fingerprint in place.

MCP tools are available to the **main agent only** — not to coding-harness child processes. Plan mode still excludes the MCP surface.

## Supported Transports

- **stdio** — subprocess-based servers
- **HTTP (streamable)** — recommended for remote servers
- **SSE** — legacy Server-Sent Events transport

## OAuth Flow (Remote Servers)

For OAuth-protected remote MCP servers (e.g., Atlassian), CometMind drives the full spec-compliant flow:

1. **Protected Resource Metadata discovery** (RFC 9728)
2. **Authorization Server Metadata discovery** (RFC 8414)
3. **Dynamic Client Registration** (RFC 7591)
4. **Authorization Code + PKCE** — CometMind owns the loopback listener at `http://localhost:1456/mcp/oauth/callback` and opens the system browser

Users just click **Connect with OAuth** — no client ID or endpoint URLs to paste.

### Token Storage

- `~/.cometmind/mcp-oauth/{serverId}.json` — access/refresh token (mode 0600)
- `~/.cometmind/mcp-oauth/{serverId}.client.json` — registered client identity + token endpoint (mode 0600)

Neither file lives in settings JSON.

### Headless Refresh

At runtime connect, tokens are auto-refreshed using stored client info — no browser needed. Only if refresh fails is the user nudged to re-run Connect.

## Management API

- `GET /api/v1/mcp/servers` — list configured servers
- `GET /api/v1/mcp/tools` — list available MCP tools
- `POST /api/v1/mcp/servers/{id}/connection-tests` — test connection
- `POST /api/v1/mcp/servers/{id}/reconnection-runs` — reconnect
- `POST /api/v1/mcp/servers/{id}/oauth-flows` — initiate OAuth

## Import

Settings UI can import Cursor-style `mcp.json` files (`{ "mcpServers": { ... } }`).

## MCP Apps

Connect advertises `io.modelcontextprotocol/ui` with `mimeTypes: ["text/html;profile=mcp-app"]`. Discovery keeps tool `_meta`. The catalog stores `AppResourceURI` from `_meta.ui.resourceUri`, then `ui/resourceUri`, then `openai/outputTemplate`. A tool whose `_meta.ui.visibility` array omits `"model"` stays off the model catalog.

`use_mcp_tool` reads that URI only for the tool it is about to call. It accepts resource content only when the MIME type starts with `text/html;profile=mcp-app`. HTML is stored in memory, keyed by session and a random app id, for about ten minutes. The model-visible tool output is one line saying the panel opened or did not. A separate `mcp_app_ready` event carries Codex-shaped `mcpAppUi` (`resourceUri`, `preferredModelDisplayMode`) plus `serverId`, `toolName`, `appId`, and `title`.

The desktop fetches `GET /api/v1/sessions/{id}/mcp-apps/{appId}` and renders the HTML in a sandboxed `srcdoc` iframe in the workspace panel. The same `resourceUri` replaces the open view. That iframe can display the page. It cannot yet operate the view: there is no guest `postMessage` bridge and no proxy for tools the app calls. See [[mcp-apps-host-alignment]].

## Library

Uses `github.com/modelcontextprotocol/go-sdk` for the MCP protocol implementation.

## References

- [[2026-07-16-cometline-agents.md]]
- [[cometline-agent-runtime]]
- [[cometline-project]]
