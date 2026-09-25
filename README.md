# @pipeworx/prnewswire

Live press releases from PR Newswire's site-wide RSS feed — headline, issuer,
industry tags, publish time and excerpt, newest first.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/prnewswire_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/prnewswire_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-prnewswire
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
