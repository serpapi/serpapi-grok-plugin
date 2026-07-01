---
name: serpapi-search
description: |
  Multi-engine search via SerpApi — one tool spanning 130+ engines: Google (web, images, news,
  shopping, maps/local, finance, flights, hotels, jobs, scholar), Bing, Yahoo, Yandex, Baidu, Naver,
  DuckDuckGo, YouTube, eBay, Amazon, Walmart, and more. Use this skill whenever the user asks to
  search the web, find articles, research a topic, look something up, find recent news, check a stock
  price, get the weather, find local businesses, compare shopping results, or search a specific engine
  by name (e.g. "search YouTube for", "search eBay for"). Returns structured JSON — prices, ratings,
  addresses, knowledge graph data — not just links and snippets.
allowed-tools:
  - mcp__serpapi__search
  - mcp__serpapi__search_table
  - mcp__serpapi__search_dashboard
  - Bash(serpapi *)
---

# serpapi search

One tool, many engines. `params.engine` selects the search vertical (Google web/images/news/shopping/
maps/finance/flights/hotels/jobs/scholar, Bing, Yahoo, Yandex, Baidu, Naver, DuckDuckGo, YouTube,
eBay, Amazon, Walmart, and 100+ more) — see [Choosing an engine](#choosing-an-engine) below.

## Tools: MCP first, CLI fallback

This plugin bundles SerpApi's hosted **MCP server**. When it's connected, prefer the native
`search` tool — it needs no local install and authenticates through the plugin's bundled config:

```json
{"name": "search", "arguments": {"params": {"q": "coffee shops", "location": "Austin, TX", "engine": "google_maps"}, "mode": "compact"}}
```

The **`serpapi` CLI** (commands below) is the fallback when the MCP isn't connected:

```bash
serpapi search engine=google_light q="coffee shops" location="Austin, TX"
```

Both read the same `SERPAPI_KEY`. See [rules/setup.md](rules/setup.md) if it isn't set.

## Quick start

```json
{"name": "search", "arguments": {"params": {"q": "best practices for React testing"}, "mode": "compact"}}
```

```json
{"name": "search", "arguments": {"params": {"q": "AAPL stock", "engine": "google_finance"}}}
```

```bash
serpapi search engine=youtube q="rust async tutorial"
```

`params.engine` defaults to `google_light` when omitted — a fast, lightweight Google web search with
clean organic results. Prefer `mode: "compact"` (MCP) or `--fields`/`--jq` (CLI, see below) unless the
full raw payload is needed — it substantially cuts context usage.

## Choosing an engine

Default to `_light` variants for AI-agent use — lower latency, smaller payload:

| Need                        | Engine                                          |
| ---------------------------- | ------------------------------------------------ |
| General web (default)          | `google_light` — use `google` only for knowledge graph / local pack / featured snippets |
| Images                        | `google_images_light`                             |
| News                           | `google_news_light`                               |
| Shopping / product prices       | `google_shopping_light`                            |
| Local businesses / maps          | `google_maps` or `google_local`                     |
| Finance / stock quotes            | `google_finance`                                   |
| Flights / hotels / jobs             | `google_flights` / `google_hotels` / `google_jobs`    |
| Academic / research                  | `google_scholar`                                     |
| Video                                 | `youtube`                                             |
| Marketplace listings                    | `ebay`, `amazon`, `walmart`                           |
| Alternative web engines                   | `bing`, `duckduckgo`, `yahoo`, `yandex`, `baidu`, `naver` |
| Weather                                     | `google` with a weather query (e.g. `"weather in London"`) |

This covers common cases only — 130+ engines exist (patents, trends, app stores, transcripts, reverse
image, etc.). Before using an unfamiliar engine or one with non-obvious parameters, check its schema
rather than guessing:

- **MCP**: read the `serpapi://engines` resource (index) or `serpapi://engines/<engine>` (full schema)
- **CLI/no MCP**: https://serpapi.com/search-engine-apis lists every engine and its parameters

## Parameters

| Parameter          | Description                                                   |
| ------------------- | -------------------------------------------------------------- |
| `params.q`           | Search query (required)                                        |
| `params.engine`       | Engine to use (default: `google_light`)                        |
| `params.location`     | Geographic location for location-sensitive results              |
| `mode`               | `"complete"` (full payload, default) or `"compact"` (condensed, MCP only) |
| other `params.*`      | Engine-specific — check the engine's schema if unsure            |

## Token efficiency

- **MCP**: pass `mode: "compact"` — strips `search_metadata`/`search_parameters`.
- **CLI**: use `--fields` (server-side, maps to SerpApi's `json_restrictor`) to drop whole sections,
  or `--jq` (client-side, embedded jq) to shape the exact output:
  ```bash
  serpapi search --fields "organic_results" engine=google_light q="query"
  serpapi search --jq '[.organic_results[]|{title,link,snippet}]' engine=google_light q="query"
  ```
- For large result sets either way, write JSON to `.serpapi/search-{query}.json` rather than keeping
  the full payload in context.

## serpapi CLI reference (fallback)

Install: `brew install serpapi/tap/serpapi-cli` (or download a binary from GitHub Releases, or
`go install github.com/serpapi/serpapi-cli/cmd/serpapi@latest`).

| Subcommand   | Purpose                                        |
| ------------ | ----------------------------------------------- |
| `search`     | Execute a query against any SerpApi engine       |
| `account`    | View account details, usage, and remaining quota  |
| `locations`  | Look up available search locations (no key needed) |
| `archive`    | Retrieve a cached result by search ID              |
| `login`      | Interactive auth setup                              |

Auth priority: `--api-key` flag > `SERPAPI_KEY` env var > `~/.config/serpapi/config.toml`. Exit codes:
`0` success, `1` API error, `2` usage error — errors are always JSON on stderr, safe to parse.

```bash
serpapi search engine=google q=coffee
serpapi search --all-pages --max-pages 3 engine=google q=coffee
serpapi account   # check remaining credits before a large batch of searches
```

## Composition patterns

- **Research fan-out** — for broad questions, query multiple engines/angles in parallel and
  synthesize: `serpapi search engine=google_finance q=... & serpapi search engine=google_news_light q=... & wait`
- **Progressive refinement** — try an exact-match query with a small `num` first; if results are
  sparse, broaden the query or switch engine (`bing`, `duckduckgo`) before giving up.
- **Verification** — cross-check a claim against two engines; agreement raises confidence, divergence
  should be surfaced to the user rather than silently resolved.

## Errors

- **401** — invalid or missing key. Re-check `SERPAPI_KEY` / the MCP auth header.
- **429** — quota exceeded. Check `serpapi account` or https://serpapi.com/dashboard before retrying;
  don't loop retries against a maxed-out quota.
- **400** — missing required parameter (`q` or `engine`).

## Interactive results (MCP Apps-capable clients only)

- `search_table` — same `params`, renders organic results as a sortable/searchable table
- `search_dashboard` — same `params`, renders summary metrics + a source-breakdown chart

## Tips

- Each call costs SerpApi credits — don't issue near-duplicate searches (same query/engine); reuse a
  prior result still in context or saved to `.serpapi/`.
- Match engine to intent: `google_news_light` for "recent news," `google_shopping_light` for "compare
  prices," `google_maps` for "near me" queries — don't default to plain `google_light` when a more
  specific engine is obviously right.

## See also

- [rules/setup.md](rules/setup.md) — getting and configuring `SERPAPI_KEY` for both MCP and CLI
- [rules/security.md](rules/security.md) — handling untrusted result content
