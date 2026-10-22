# SerpApi Plugin for Grok Build

This plugin gives Grok Build the ability to search Google, Bing, YouTube, eBay, Amazon, Walmart, Google Maps, Google Scholar, and 130+ other engines through SerpApi, and to read web pages with the SerpApi Extract API.

## Quick Start

### 1. Get an API key

Sign up at [serpapi.com](https://serpapi.com/dashboard) and set your API key.

```bash
export SERPAPI_KEY="your-key-here"
```

Unlike OAuth based plugins, SerpApi's hosted MCP server authenticates with this bearer token, so the key must be set before Grok Build starts.

### 2. Install the plugin

Install it directly from the [serpapi-grok-plugin](https://github.com/serpapi/serpapi-grok-plugin) repository.

```bash
grok plugin install serpapi/serpapi-grok-plugin --trust
```

This wires up the bundled MCP server and two skills: `serpapi-search` and `serpapi-extract`. Once the plugin is listed in a marketplace source, you'll also be able to find it in Grok Build by running `/plugin` and searching for **serpapi**.

See [skills/serpapi-search/rules/setup.md](skills/serpapi-search/rules/setup.md) for detailed setup and troubleshooting.

### 3. Use it

Once installed and authenticated, ask naturally.

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

```text
Summarize https://serpapi.com/blog/using-serpapi-in-google-colab/
```

## Features

The hosted MCP server is the primary way this plugin works. Its native `search` tool is the default execution path, authenticated with the bearer key you provide.

- **Single tool, all engines.** The `serpapi-search` skill covers 130+ engines through one `search` tool, with `params.engine` selecting the vertical.
- **Read web pages.** The `serpapi-extract` skill reads pages through the Extract API (`engine: "extract"` on the same `search` tool): scrape a URL into structured content and Markdown, map a site's URLs, or crawl several pages.
- **Structured results.** Responses are JSON with prices, ratings, addresses, and other fields, not just links and snippets.
- **Interactive views.** MCP Apps capable clients can render results with `search_table`, a sortable table, and `search_dashboard`, a metrics dashboard.
- **Fallback friendly.** The [SerpApi CLI](https://github.com/serpapi/serpapi-cli) is available as an optional fallback for when the MCP server is unavailable. It reads the same `SERPAPI_KEY`, so no separate authentication step is needed.

```bash
brew install serpapi/tap/serpapi-cli
```

## Supported Engines

| Category     | Engines                                                             |
| ------------ | --------------------------------------------------------------------- |
| Web Search   | Google, Bing, DuckDuckGo, Yahoo, Yandex, Baidu, Naver                   |
| Shopping     | Amazon, Walmart, eBay, Google Shopping                                  |
| Local / Maps | Google Maps, Google Local                                              |
| Research     | Google Scholar, Google Patents, Google Trends                           |
| News         | Google News, Bing News, DuckDuckGo News, Baidu News                      |
| Media        | Google Images, Google Videos, YouTube                                    |
| Travel       | Google Flights, Google Hotels                                            |
| Jobs         | Google Jobs                                                              |
| Finance      | Google Finance                                                           |
| Web Pages    | Extract (scrape, map, crawl)                                             |

See the full list of [Supported Engines](https://serpapi.com/search-engine-apis).

### CLI Commands (optional fallback)

| Command                 | Description                                          |
| ------------------------ | ------------------------------------------------------ |
| `serpapi search ...`     | Run a search against any engine                          |
| `serpapi search engine=extract url=...` | Read a web page with the Extract API (`mode=scrape`, `map`, or `crawl`) |
| `serpapi image <file>`   | Upload an image for Google Lens and print its `image_id` (CLI v0.4.0+) |
| `serpapi account`        | Check remaining quota and credits                          |
| `serpapi locations`      | Look up available search locations, no key needed          |
| `serpapi archive <id>`   | Retrieve a cached result by search ID                        |
| `serpapi login`          | Interactive authentication setup                              |

Extract has no subcommand of its own; it's an engine of `serpapi search`:

```bash
serpapi search engine=extract url=https://example.com include_markdown=true --jq '.text_content.markdown'
serpapi search engine=extract url=https://serpapi.com mode=map --fields 'urls[].url'
```

## Output Files

For large result sets, results are saved to a `.serpapi/` directory in your project to keep Grok Build's context window clean.

```text
.serpapi/search-coffee_shops_austin.json
.serpapi/search-react_testing.json
```

## Configuration

| Variable      | Required | Description                                                          |
| ------------- | -------- | ---------------------------------------------------------------------- |
| `SERPAPI_KEY`  | Yes      | Bearer key used by both the MCP `Authorization` header and the CLI fallback |

## Troubleshooting

- **Invalid API key.** Verify your key at [serpapi.com/manage-api-key](https://serpapi.com/manage-api-key).
- **MCP tools not available.** Start a new Grok Build session after installing the plugin or changing MCP configuration.
- **Plugin not listed.** Run `grok plugin list`, then reinstall it if it's missing.
- **Quota or rate limit exceeded.** Check usage in the [SerpApi dashboard](https://serpapi.com/dashboard) or review [pricing](https://serpapi.com/pricing).

Run `grok mcp doctor serpapi` for a detailed diagnosis of MCP connection issues.

## Testing

See [TEST.md](TEST.md) for how to validate and exercise this plugin locally against the `grok` CLI.

## Resources

- [SerpApi Documentation](https://serpapi.com/search-api)
- [SerpApi MCP Server Repository](https://github.com/serpapi/serpapi-mcp), the MCP server integration used by this plugin and other MCP compatible clients.
- [SerpApi CLI Repository](https://github.com/serpapi/serpapi-cli), the optional command line fallback.
- [Supported Engines](https://serpapi.com/search-engine-apis)
- [Get API Key](https://serpapi.com/dashboard)

## License

This plugin is licensed under MIT.

## Support

Report issues at [SerpApi MCP GitHub Issues](https://github.com/serpapi/serpapi-mcp/issues).
