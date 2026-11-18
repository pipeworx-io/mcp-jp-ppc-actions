# @pipeworx/jp-ppc-actions

Has the Japan Personal Information Protection Commission (個人情報保護委員会)
announced an administrative response (行政上の対応: guidance, recommendation or
order) against this company under the Act on the Protection of Personal
Information, for example after a data breach?

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_ppc_actions_search(name, limit?)` — exact or partial company-name match after
  Unicode folding and dropping corporate-form words. Each match gives the
  announcement date, the companies named, the announcement's title, whether it is a
  follow-up progress report on an earlier action, whether it was taken under the
  My Number Act, and a link to the commission's announcement. Also returns `result`
  (`hit` / `no_hit`), `total_matches`, `notices_read`, `covers`, `unnamed_notices`,
  `omitted_non_company` and `data_as_of`.
- `jp_ppc_actions_read(link)` — one announcement: the date as the commission prints
  it, its heading and the link, format and size of the attached document that
  states what the commission did.

## Auth

Keyless.

## Data sources

- Japan Personal Information Protection Commission (個人情報保護委員会), press
  announcements of administrative responses, from fiscal 2021 (April 2021) onward.
  Read from the publisher on request and held in the serving process for an hour.
  The commission publishes no table, so each response is read from the title of its
  announcement, which names the company. `data_as_of` is the day the lists were
  read; `covers.to` is the latest announcement.

## Behaviour worth knowing

- Whether a response was guidance, a recommendation or an order is stated only in the
  attached document, which is not read; every response says so (`response_kind`).
- Titles that name no company ("insurers and agencies", "opt-out notified businesses")
  cannot be searched by name and are counted in `unnamed_notices`. Public bodies
  (municipalities, boards of education, agencies) and individuals are not served and
  are counted in `omitted_non_company`; the commission publishes no corporate numbers.
- A list page without the expected markup, a year that parses to nothing or an HTTP
  failure is refused, so a source that is down is never read as "no action found".
- No hit means no announcement naming that company in the period covered, as read now.
  It does not clear the company.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-ppc-actions": {
      "url": "https://gateway.pipeworx.io/jp-ppc-actions/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-ppc-actions/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_ppc_actions_search \
  -H 'Content-Type: application/json' \
  -d '{"name":"株式会社イセトー"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_ppc_actions_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-ppc-actions` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-ppc-actions/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-ppc-actions": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-ppc-actions"]
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
ask_pipeworx({ question: "your question about Jp Ppc Actions data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
