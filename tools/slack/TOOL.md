---
name: slack
description: Slack official MCP server
url: https://mcp.slack.com/mcp
---

# slack

OAuth only — no fallback header. Each project connects with its own Slack user
token, so what a run can see is what the connecting user can see.

Note the address and the identity are not the same string: the server is served
at `https://mcp.slack.com/mcp` but identifies itself as `https://mcp.slack.com`,
which is the `resource` every token is bound to. Discover reads that off the
server's own metadata — do not hand-edit it.

Sync creates the entry without the OAuth block; run Discover after the first sync.
