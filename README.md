# @pipeworx/prnewswire

Live press releases from PR Newswire's site-wide RSS feed — headline, issuer,
industry tags, publish time and excerpt, newest first.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `prnewswire_latest_releases(since_hours?, limit?)` — the current rolling
  window of releases, newest first. `newest_item_at` tells a caller whether
  the feed is quiet or stalled.
- `prnewswire_search_releases(query, limit?)` — keyword/company match over
  headline, issuer, excerpt and industry tags, within the current window only.
- `prnewswire_get_release(url)` — one specific release, matched by the URL a
  prior call returned. Fails clearly (not silently empty) if the release has
  scrolled out of the current window.

## Auth

Keyless. No account, no clickthrough terms.

## Data sources

- <https://www.prnewswire.com/rss/news-releases-list.rss> — PR Newswire's
  "All News Releases" feed. Its own description spans politics/government,
  business, technology, religion, sports/entertainment, science/nature and
  health/lifestyle, in English or other languages. It holds roughly the
  ~20 most recent releases site-wide — typically under two hours of volume —
  and every call re-fetches this same rolling window live. It is **not** an
  archive: `search_releases` only ever searches what is currently in the
  window, and `get_release` can miss a release that has already scrolled off.

The issuer name is pulled from `<dc:contributor>`, not from
`<media:credit role="publishing company">` — that tag names PR Newswire
itself on every item, not the issuer, so treating it as a byline would
mislabel every release as being "by PRNewswire".

**Cloudflare bot-management quirk, worth knowing if you extend this pack:**
the feed URL intermittently 301-redirects to a trailing-slash variant that
itself 404s — roughly 1 in 3 fresh requests, measured live 2026-09-21
(8-request sample: 3/8). It is a cookie-issuing challenge redirect, not a
real outage; the next fresh request typically succeeds. This pack retries up
to 3 attempts before reporting the wire down.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "prnewswire": {
      "url": "https://gateway.pipeworx.io/prnewswire/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/prnewswire/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/prnewswire_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/prnewswire_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-prnewswire` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/prnewswire/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "prnewswire": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-prnewswire"]
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
ask_pipeworx({ question: "your question about Prnewswire data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
