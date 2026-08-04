---
name: Authorize a DataGol MCP connector and execute a tool
description: Walk the Events.com DataGol MCP connector OAuth flow, list the tools a connection exposes, mint a per-user MCP URL, and execute a tool.
api: openapi/eventscom-datagol-platform-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - listConnectors
  - connect
  - oauthCallback
  - getConnections
  - getConnectionById
  - getToolsForConnection
  - getMcpServerUrl
  - executeTool
  - updateConnectionName
  - deleteConnection
---

# Authorize a DataGol MCP connector and execute a tool

DataGol brokers third-party tool access through **Composio**-backed MCP connectors. The flow is: pick a
connector → run OAuth → get a connection → enumerate its tools → either execute a tool server-side or mint
a per-user MCP URL for an agent client to attach to.

## Before you start

- **Auth.** `Authorization: Bearer <jwt>` (`s_jwt`) on `https://datagol-be.events.com/`.
- **Two different "MCP" things live here.** These operations are the *control plane* that configures
  connectors. Events.com's own hosted MCP server is a separate host, `https://datagol-mcp.events.com/`,
  which gates every request on `workspace_id`, `workbook_id` and `token` before the JSON-RPC layer.
  See `mcp/eventscom-mcp.yml`.
- **No OAuth discovery.** `/.well-known/oauth-authorization-server` and
  `/.well-known/oauth-protected-resource` are not served, so a standards-based MCP client cannot
  auto-discover authorization. Configure it manually.

## Steps

1. **List available connectors** — `listConnectors`
   `GET /mcpConnectorAuth/api/v1/connectors`

2. **Start the OAuth handshake** — `connect`
   `POST /mcpConnectorAuth/api/v1/connect` → redirect the user to the returned authorization URL.

3. **Completion is server-side** — `oauthCallback`
   `GET /mcpConnectorAuth/api/v1/callback` is the redirect target Composio calls. Do not invoke it
   yourself; just wait for the connection to appear.

4. **Confirm the connection landed** — `getConnections`
   `GET /mcpConnectorAuth/api/v1/connections`, or `getConnectionById`
   (`GET /mcpConnectorAuth/api/v1/connections/{connectionId}`).

5. **Enumerate the tools that connection grants** — `getToolsForConnection`
   `GET /mcpConnectorAuth/api/v1/connections/{connectionId}/tools`
   This is the authoritative tool list for the connection's toolkit. Read it rather than assuming a tool
   exists.

6. **Then either —**

   **(a) Execute server-side** — `executeTool`
   `POST /mcpConnectorAuth/api/v1/tools/execute`
   This is an **acting**, third-party-side-effecting call with no idempotency key. Do not auto-retry on
   timeout; re-read state at the provider first.

   **(b) Hand an MCP URL to an agent client** — `getMcpServerUrl`
   `GET /mcpConnectorAuth/api/v1/mcpServers/{serverId}/url`
   Mints a **per-user** MCP URL. Treat it as a credential: it embeds the user's authorization, so never log
   it, never share it across users, and scope its lifetime.

7. **Housekeeping** — `updateConnectionName`
   (`PATCH /mcpConnectorAuth/api/v1/connections/{connectionId}`) and `deleteConnection`
   (`DELETE /mcpConnectorAuth/api/v1/connections/{connectionId}`).

## Connector configuration (separate group)

`Agent MCP` and `agent-mcp-config-controller` manage the connector records themselves:
`getAll_2` / `create_5` / `get_3` / `update_3` / `delete_5` on `/mcp/api/v1`, plus `getToolkits` and
`updateToolkits` on `/mcp/api/v1/{mcpId}/toolkits`, and `listMcpConfigs` / `createMcpConfig` /
`getMcpConfig` / `updateMcpConfig` / `deleteMcpConfig` on `/mcpConfigs/api/v1`.

`initiateMcpAuth` (`POST /connector/api/v1/mcp/credential`) and `mcpCallback`
(`GET /connector/api/v1/mcp/callback`) are a parallel OAuth2 credential-registration path.

See `mcp/eventscom-tool-crosswalk.yml`.
