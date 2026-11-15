# @pipeworx/acra-sg

Singapore's register of entities with a Unique Entity Number (UEN): look up a company, business or foreign branch by name or UEN and get its legal name, status, entity type, registration date and registered street and postal code.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1957+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `acra_company_search(name | uen, status?, limit?)` — name search matches entities whose name contains every word given; best matches first (exact name, then name prefix, Registered before Deregistered). An empty result returns `found:false` with `empty_reason`.
- `acra_company_get(uen)` — one entity by UEN. An unknown UEN returns `found:false` with a `reason`.

Every response carries `data_as_of` (the publisher's last monthly update) and `source`.

## Auth

Keyless.

## Data sources

- Accounting and Corporate Regulatory Authority (ACRA), Singapore: "Entities Registered with ACRA", published as open data. Updated monthly by the publisher, so an entity registered this month may not appear yet.

The open data does not include officers, directors, shareholders, business activity codes or unit-level addresses (the publisher removed those columns), so none are returned. English-only register: `legal_name_native` is always null.

Status is the register's own word (for example "Registered", "Deregistered").

## Notes

- Name matching is word-based, so "Grab Holdings" returns every entity with both words, not the listed company of that name; add more words or use the UEN.
- The status filter is applied to the best 100 name matches, not the whole register; the response says when results were truncated.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "acra-sg": {
      "url": "https://gateway.pipeworx.io/acra-sg/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/acra-sg/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1957+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/acra_company_search \
  -H 'Content-Type: application/json' \
  -d '{"name":"Singapore Airlines","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/acra_company_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-acra-sg` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/acra-sg/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "acra-sg": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-acra-sg"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Acra Sg data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
