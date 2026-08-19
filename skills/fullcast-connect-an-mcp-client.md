---
name: Connect an AI client to the Fullcast MCP server
description: Complete the OAuth 2.1 authorization-code + PKCE flow against Fullcast's MCP endpoint and list the tools available to the tenant.
api: https://app.fullcast.io/mcp
transport: streamable-http
operations: [register_client_oauth_register_post, authorize_oauth_authorize_get, token_exchange_oauth_token_post]
mcp_tools: [ping, get_server_info]
---

# Connect an AI client to the Fullcast MCP server

Fullcast exposes its go-to-market platform to agents over MCP. Both endpoints are OAuth-gated; there is no API key path and no stdio package.

## Preconditions

The tenant admin must first enable AI features, or every call returns 401 no matter how good your token is:

> Settings > Application Settings > AI & Agents > **Enable AI Assistant Features**

This is per tenant and admin-only. See `https://support.fullcast.com/docs/enable-ai-features.md`.

## Pick the endpoint

| Server | Endpoint | Tools |
|---|---|---|
| Fullcast MCP (primary) | `https://app.fullcast.io/mcp` | ~88 documented |
| Fullcast Assistant MCP | `https://assistant.fullcast.io/mcp/` | 17 verified |

Use the primary unless you specifically need the assistant server's guidance resources.

## Steps

1. **Discover the authorization server.** POST anything to the MCP endpoint unauthenticated. The 401 carries the pointer:

   ```
   WWW-Authenticate: Bearer realm="mcp-openapi",
     resource_metadata="https://app.fullcast.io/.well-known/oauth-protected-resource/mcp"
   ```

   Fetch that, then fetch `https://app.fullcast.io/mcp/.well-known/oauth-authorization-server` (RFC 8414). Do **not** request `/.well-known/openid-configuration` — Fullcast returns `{"error":"oidc_not_supported"}` on purpose.

2. **Register a client** (`registration_endpoint`, RFC 7591). `redirect_uris` must be an **array** or you get `400 invalid_client_metadata`.

3. **Authorize** at `https://app.fullcast.io/mcp/authorize` with `response_type=code` and PKCE. The primary server supports **`S256` only**; the assistant server also accepts `plain` — always use `S256`.

4. **Exchange the code** at `https://app.fullcast.io/mcp/token`. `token_endpoint_auth_methods_supported` is `["client_secret_post","none"]`, so a public client is fine.

5. **Call the server** with `Authorization: Bearer <token>` and `Accept: application/json, text/event-stream`. Start with `tools/list`, then `ping` and `get_server_info`.

## Scopes

The assistant issuer publishes `openid`, `email`, `profile`, `offline_access`, `mcp:tools`, `mcp:resources`, `mcp:prompts`. Request `offline_access` if the agent runs unattended.

**`mcp:tools` is not capability-scoped.** One scope covers reads *and* writes — the same token that calls `get_account` can call `move_account` and `end_coverage`. There is no read-only scope. Gate destructive tools in your own client.
