# Testing the SerpApi plugin in Grok Build

Requires `grok` CLI **1.0.13 or later** (tested on 1.0.62; run `grok update`) and a SerpApi API key —
the free plan works: https://serpapi.com/manage-api-key

## 1. Validate the plugin

```bash
git clone https://github.com/serpapi/serpapi-grok-plugin.git
grok plugin validate ./serpapi-grok-plugin
```

Expected:

```
Plugin manifest is valid.
  name: serpapi
  version: 0.2.0
  ...
  components: 1 skill dir(s), 0 command dir(s), 0 agent dir(s), MCP servers
```

## 2. Add the PR branch as a marketplace source

Add to `~/.grok/config.toml`:

```toml
[[marketplace.sources]]
name = "serpapi-pr"
git = "https://github.com/strzibny/plugin-marketplace.git"
branch = "add-serpapi-plugin"
```

```bash
grok plugin marketplace update serpapi-pr   # → serpapi-pr: synced
```

## 3. Install from the catalog

```bash
grok plugin install serpapi --trust   # → Installed 1 plugin(s) from serpapi-pr: serpapi
grok plugin details serpapi
grok inspect | grep serpapi   # → serpapi-extract, serpapi-search, serpapi (http)
```

Expected: `kind: git: https://github.com/serpapi/serpapi-grok-plugin.git`, `source: serpapi-pr`,
`serpapi v0.2.0`, `1 skill dir(s)`, `MCP servers`. Grok installs the commit pinned in the PR's
`.grok-plugin/marketplace.json`.

## 4. Check the MCP connection

```bash
grok mcp doctor serpapi
```

Expected: `serpapi (http: https://mcp.serpapi.com/mcp)` with `✓ handshake OK` and
`✓ 3 tools discovered` (`search`, `search_table`, `search_dashboard`). The handshake doesn't need a key, so
step 5 is the real check that the key works.

## 5. Run a live search

The key must be exported before Grok starts. The plugin's `.mcp.json` sends it as
`Authorization: Bearer ${SERPAPI_KEY}`.

```bash
export SERPAPI_KEY="your-key"
grok -p "Search Google with serpapi for 'SerpApi' and list the top 3 organic results with title and URL." \
  --allow serpapi__search
```

`--allow` is needed because headless runs can't prompt for tool approval. Grok exposes MCP tools as
`<server>__<tool>`, so the plugin's `search` tool is `serpapi__search`.

Expected: Grok reads the `serpapi-search` skill, calls `serpapi__search` (e.g. `engine: google_light`,
`q: SerpApi`) and lists three results with titles and URLs.

Web page extraction (Extract API) through the same tool:

```bash
grok -p "Use SerpApi to get https://example.com as Markdown and tell me the page title and first sentence." \
  --allow serpapi__search
```

Expected: Grok reads the `serpapi-extract` skill and makes a `serpapi__search` call with `engine: extract`, `include_markdown: true`, and an answer of
**Example Domain** / *"This domain is for use in documentation examples without needing permission."*

To try more engines, run `grok` interactively and approve the tool call when prompted, e.g.:

- *"What's the current AAPL stock price?"* (Google Finance)
- *"Top-rated coffee shops near Union Square, SF"* (Google Maps)
- *"Cheapest flights SFO→JFK next Friday"* (Google Flights)

Without a valid key the tool returns:
`Invalid SerpApi API key. Check the key in the request path or Authorization header...`

> If you already have a SerpApi MCP server configured elsewhere (e.g. in `~/.claude.json`), Grok loads it
> too. Prefix commands with `GROK_CLAUDE_MCPS_ENABLED=false` so only the plugin's server is used.

## 6. Clean up

```bash
grok plugin uninstall serpapi
```

Then remove the `serpapi-pr` block from `~/.grok/config.toml`.

## Network and credentials

The plugin calls only `https://mcp.serpapi.com/mcp` and needs only `SERPAPI_KEY`. It has no hooks,
commands or agents.
