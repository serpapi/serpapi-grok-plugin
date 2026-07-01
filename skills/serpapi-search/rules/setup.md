---
name: serpapi-setup
description: |
  Get and configure a SerpApi API key so the bundled hosted MCP server and the serpapi-cli
  fallback can both authenticate.
  Docs: https://serpapi.com/dashboard
---

# SerpApi Setup

Both the bundled MCP server (`mcp.serpapi.com`) and the `serpapi-cli` fallback authenticate with the
same bearer key — there's no OAuth flow, so each user must supply their own.

## 1. Get an API key

Sign up and copy a key from https://serpapi.com/dashboard.

## 2. Set the environment variable

```bash
export SERPAPI_KEY="your-key-here"
```

Add this to your shell profile (`~/.zshrc`, `~/.bashrc`) so it persists across sessions.

- The plugin's `.mcp.json` reads it via `${SERPAPI_KEY}` in the `Authorization: Bearer` header.
- `serpapi-cli` reads the same variable directly, or accepts `--api-key`, or run `serpapi login` for
  an interactive setup that writes `~/.config/serpapi/config.toml`.

## 3. Install the CLI fallback (optional, but recommended)

```bash
brew install serpapi/tap/serpapi-cli
```

Or download a binary from GitHub Releases, or `go install github.com/serpapi/serpapi-cli/cmd/serpapi@latest`.
The MCP tool is preferred when connected; the CLI is only needed as a fallback.

## 4. Verify

Reconnect the MCP (or restart the session) and check connection status, then run one small request:

```json
{"name": "search", "arguments": {"params": {"q": "test query"}, "mode": "compact"}}
```

```bash
serpapi search engine=google_light q="test query"
serpapi account   # confirms auth and shows remaining quota
```

## If authentication fails

- Confirm `echo $SERPAPI_KEY` prints a non-empty value in the same shell the agent runs in.
- Confirm the key is active and has remaining credits via `serpapi account` or
  https://serpapi.com/dashboard.
- As an alternative to the header form, SerpApi also supports putting the key directly in the MCP
  URL path (`https://mcp.serpapi.com/YOUR_API_KEY/mcp`) — only do this if the header approach isn't
  working, since it's more likely to leak the key into logs or shared config.
