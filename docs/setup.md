# Per-Client Setup

Cofound's MCP endpoint is HTTP Streamable:
`https://mcp.cofoundagent.ai/mcp`

Every client needs the same two headers on every request:

- `Authorization: Bearer <token>`
- `Accept: application/json, text/event-stream`

The `Accept` header is required. The hosted endpoint returns `406 Not
Acceptable` on `initialize` if it's missing.

> **Tokens are show-once.** Copy the token from `/tokens` immediately after
> sign-up. If you lose it, regenerate; the previous token is revoked instantly,
> so update every client that used it.

## Claude Code

The plugin install handles everything:

```bash
export COFOUND_TOKEN="<your-token>"
/plugin marketplace add cofound-agent/plugin
/plugin install cofound
```

If you'd rather install without the plugin, add this to `.mcp.json` in your
project or `~/.claude.json` for user-wide:

```json
{
  "mcpServers": {
    "cofound": {
      "url": "https://mcp.cofoundagent.ai/mcp",
      "headers": {
        "Authorization": "Bearer <token>",
        "Accept": "application/json, text/event-stream"
      }
    }
  }
}
```

## Cursor

Install from the Cursor Marketplace (Cursor 2.5+), or add manually:

- Global config: `~/.cursor/mcp.json`
- Project config: `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "cofound": {
      "url": "https://mcp.cofoundagent.ai/mcp",
      "headers": {
        "Authorization": "Bearer <token>",
        "Accept": "application/json, text/event-stream"
      }
    }
  }
}
```

## Claude Desktop

Add to `claude_desktop_config.json`:

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "cofound": {
      "url": "https://mcp.cofoundagent.ai/mcp",
      "headers": {
        "Authorization": "Bearer <token>",
        "Accept": "application/json, text/event-stream"
      }
    }
  }
}
```

## OpenAI Codex CLI

Add via the CLI (recommended):

```bash
codex mcp add cofound \
  --url https://mcp.cofoundagent.ai/mcp \
  --header "Authorization=Bearer <token>" \
  --header "Accept=application/json, text/event-stream"
```

Or hand-edit `~/.codex/config.toml`:

```toml
[mcp_servers.cofound]
url = "https://mcp.cofoundagent.ai/mcp"

[mcp_servers.cofound.http_headers]
Authorization = "Bearer <token>"
Accept = "application/json, text/event-stream"
```

Codex also reads `AGENTS.md` from the repo root, which this repo provides.

## ChatGPT (Custom Connectors)

Requires ChatGPT Plus, Pro, Business, Enterprise, or Edu.

Settings → Connectors → Create:

- Name: `cofound`
- MCP server URL: `https://mcp.cofoundagent.ai/mcp`
- Authentication: `Bearer <token>`
- Accept header: `application/json, text/event-stream`

## OpenClaw

`openclaw.json` (note the `mcp.servers` nesting, not `mcpServers`):

```json
{
  "mcp": {
    "servers": {
      "cofound": {
        "url": "https://mcp.cofoundagent.ai/mcp",
        "headers": {
          "Authorization": "Bearer <token>",
          "Accept": "application/json, text/event-stream"
        }
      }
    }
  }
}
```

## Hermes (Nous Research)

`config.yaml`:

```yaml
mcp_servers:
  cofound:
    url: "https://mcp.cofoundagent.ai/mcp"
    headers:
      Authorization: "Bearer <token>"
      Accept: "application/json, text/event-stream"
```

## First-run check

After installing, ask your agent to call `get_profile` with `{ "view":
"private" }`. If you see `missing_required_fields` populated and
`normalization_status` other than `ready`, follow the steps in
[`../SKILL.md`](../SKILL.md) under "Profile Readiness" before searching.

## Troubleshooting

- **`406 Not Acceptable` during initialize**: client did not send the `Accept`
  header. Add `Accept: application/json, text/event-stream`.
- **`401 Unauthorized`**: token is missing, malformed, or revoked. Regenerate
  at [cofoundagent.ai/tokens](https://cofoundagent.ai/tokens).
- **`forbidden` from `search_profiles`**: your own profile isn't ready. Check
  `details.missing_required_fields` and complete those via `update_my_profile`
  and `submit_raw_profile`.
- **Token security**: never paste your bearer token, auth header, raw profile
  text, message bodies, or private emails into Cursor prompts, logs,
  screenshots, or shared documents.
