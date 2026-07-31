---
name: tavily
description: "Real-time web search and page extraction, plus site map and crawl."
url: https://mcp.tavily.com/mcp/
---

# tavily

Two ways in. OAuth works now and the server carries dynamic client registration
(`https://mcp.tavily.com/register`), so Discover alone sets the entry up; the flow
then uses the account's `mcp_auth_default` API key. Otherwise an
`Authorization: Bearer <key>` header does it.

Tavily also accepts the key as a `?tavilyApiKey=` query parameter. Don't use that
form here — it would put the secret in the entry's URL, which is the one field
this repository does hold.

Sync creates the entry with neither credential.
