---
name: agent-store
description: "Korean public and corporate data through one gateway: filings and financials, BOK and KOSIS statistics, legislation, procurement, real estate."
url: https://agent.store/mcp
---

# agent-store

Two ways in. OAuth is the documented default and the server carries dynamic
client registration (`https://agent.store/oauth/register`, scopes `mcp` and
`offline_access`), so the console's Discover step is enough to set the entry up —
nothing to register by hand. An `X-API-Key` header still works where the browser
flow is impractical.

Sync creates the entry with neither. Run Discover after the first sync, or paste
the key in the console.

Connection docs: https://agent.store/docs/connect
