# SerpApi Plugin for Grok Build

Real-time, multi-engine search results — directly from Grok Build.

This plugin bundles SerpApi's hosted [MCP server](https://github.com/serpapi/serpapi-mcp) plus a
skill that documents how to use it, giving Grok Build structured search across 130+ engines: Google
(web, images, news, shopping, maps/local, finance, flights, hotels, jobs, scholar), Bing, Yahoo,
Yandex, Baidu, Naver, DuckDuckGo, YouTube, eBay, Amazon, Walmart, and more.

**The hosted MCP is the primary way this plugin works** — its native `search` tool is the default
execution path, authenticated via a bearer key you provide. The [SerpApi CLI](https://github.com/serpapi/serpapi-cli)
is an optional fallback, used when the MCP is unavailable.

## Features

- **Search** — one `search` tool spanning 130+ engines via `params.engine`
- **Structured results** — JSON with prices, ratings, addresses, and other fields, not just links
  and snippets
- **Interactive views** (MCP Apps-capable clients only) — `search_table` and `search_dashboard`
  render results as a sortable table or a metrics dashboard

## Installation

### 1. Install the Plugin

In Grok Build, run `/plugin` and search for **serpapi**, then select it to install. This wires up
the bundled MCP server and the `serpapi-search` skill.

### 2. Get and set your API key

Unlike OAuth-based plugins, SerpApi's hosted MCP authenticates with a bearer token you provide:

```bash
export SERPAPI_KEY="your-key-here"
```

Get a key at https://serpapi.com/dashboard. See
[skills/serpapi-search/rules/setup.md](skills/serpapi-search/rules/setup.md) for details and
troubleshooting.

### Optional: SerpApi CLI (fallback)

The bundled MCP covers normal search use. For when the MCP isn't connected, install the
[SerpApi CLI](https://github.com/serpapi/serpapi-cli):

```bash
brew install serpapi/tap/serpapi-cli
```

It reads the same `SERPAPI_KEY`, so no separate auth step is needed.

## Usage

Once installed and authenticated, ask naturally:

```text
Search for "best practices for React testing" and compile the key recommendations
```

```text
Find coffee shops near Austin, TX
```

```text
What's the AAPL stock price right now?
```

```text
Search YouTube for "rust async tutorial"
```

### CLI Commands (optional fallback)

| Command             | Description                                     |
| -------------------- | ------------------------------------------------ |
| `serpapi search ...`   | Run a search against any engine                   |
| `serpapi account`      | Check remaining quota/credits                       |
| `serpapi locations`    | Look up available search locations (no key needed)    |
| `serpapi archive <id>` | Retrieve a cached result by search ID                 |
| `serpapi login`        | Interactive auth setup                                 |

## Output Files

For large result sets, results are saved to a `.serpapi/` directory in your project to keep Grok
Build's context window clean:

```text
.serpapi/search-coffee_shops_austin.json
.serpapi/search-react_testing.json
```

## Configuration

| Variable      | Required | Description                                                          |
| ------------- | -------- | ---------------------------------------------------------------------- |
| `SERPAPI_KEY`  | Yes      | Bearer key used by both the MCP `Authorization` header and the CLI fallback |

## Resources

- [SerpApi Documentation](https://serpapi.com/search-api)
- [SerpApi MCP Server Repository](https://github.com/serpapi/serpapi-mcp)
- [SerpApi CLI Repository](https://github.com/serpapi/serpapi-cli)
- [Supported Engines](https://serpapi.com/search-engine-apis)
- [Get API Key](https://serpapi.com/dashboard)

## License

This plugin is licensed under MIT.

## Support

- [GitHub Issues](https://github.com/serpapi/serpapi-mcp/issues)
