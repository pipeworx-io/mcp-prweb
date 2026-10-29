# @pipeworx/prweb

Live press releases from PRWeb's site-wide "All News Releases" RSS feed —
headline, issuer, industry and subject tags, publish time and excerpt, newest
first. PRWeb is Cision's small-business wire: issuers are mostly local
businesses, nonprofits, agencies and startups rather than listed companies.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `prweb_latest_releases(since_hours?, limit?)` — the current rolling window
  of releases, newest first. `newest_item_at` tells a caller whether the feed
  is quiet or stalled.
- `prweb_search_releases(query, limit?)` — keyword/company match over
  headline, issuer, excerpt and industry tags, within the current window only.
- `prweb_get_release(url)` — one specific release, matched by the URL a prior
  call returned. Fails clearly (not silently empty) if the release has
  scrolled out of the current window.

## Auth

Keyless. No account, no clickthrough terms. `prweb.com/robots.txt` does not
disallow `/rss/`.

## Data sources

- <https://www.prweb.com/rss/news-releases-list.rss> — PRWeb's "All News
  Releases" feed. Measured 2026-09-22: 20 items spanning about 7.5 hours.

Every call re-fetches this same rolling ~20-item window live — it is **not**
an archive. `search_releases` only ever searches what is currently in the
window and `get_release` can miss something that has already fallen out of
it (the tool says so explicitly rather than returning an empty result).

Two URL traps met while finding this feed: the legacy `/rss2/daily.xml` path
301s here, and `/rss` (no slash) 301s to `/rss/`, which is a 152 KB HTML
index page that answers HTTP 200 — a 200 that is not a feed. That index is
rendered client-side, so its per-category feeds are not enumerable from the
served HTML and this pack does not read them.

The issuer is pulled from `<dc:contributor>`. `<media:credit>` names
PRNewswire itself on every item (PRWeb runs on the PR Newswire platform), so
a generic byline reader would mislabel every release.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "prweb": {
      "url": "https://gateway.pipeworx.io/prweb/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/prweb/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/prweb_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/prweb_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-prweb` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/prweb/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "prweb": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-prweb"]
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
ask_pipeworx({ question: "your question about Prweb data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
