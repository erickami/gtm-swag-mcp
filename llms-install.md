# Installing the GTM Swag MCP server

GTM Swag is a hosted, remote MCP server. There is nothing to clone, build or run locally.

1. Add a Streamable HTTP server named `gtm-swag` with the URL `https://gtmswag.com/mcp`. In a JSON MCP settings file:

   ```json
   { "mcpServers": { "gtm-swag": { "type": "http", "url": "https://gtmswag.com/mcp" } } }
   ```

2. Connect. The server answers unauthenticated requests with `401` and OAuth metadata, so the client opens a browser sign-in. The user signs in to GTM Swag (email code or Google) and chooses which permissions to grant. No API key is needed.
3. If the client cannot do OAuth, the user creates a workspace key in GTM Swag under Integrations, For developers, stores it in an environment variable named `GTM_SWAG_KEY`, and the client sends `Authorization: Bearer` followed by that variable's value, using `https://gtmswag.com/mcp?profile=gifting`. Never ask the user to paste a key into the conversation.
4. Verify by calling `get_workflow_guide`. It returns the workspace `mode`: `test` means everything works end to end but nothing ships and no real money moves.

The user needs a GTM Swag account, free at https://gtmswag.com/login. Delivery is to US addresses. Full guide: https://gtmswag.com/guides/connect.md
