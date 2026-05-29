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

One-click: export your token, then open
[`cursor-deeplink.md`](./cursor-deeplink.md) and click "Add to Cursor". Once the
plugin is published to the Cursor Marketplace (Cursor 2.5+), you'll also be able
to install it from there.

Or add the server manually:

- Global config: `~/.cursor/mcp.json`
- Project config: `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "cofound": {
      "url": "https://mcp.cofoundagent.ai/mcp",
      "headers": {
        "Authorization": "Bearer ${env:COFOUND_TOKEN}",
        "Accept": "application/json, text/event-stream"
      }
    }
  }
}
```

Cursor interpolates `${env:VAR}` in `mcp.json`, so exporting `COFOUND_TOKEN`
keeps the raw token out of the file. Paste a literal `Bearer <token>` instead if
you prefer.

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

Recommended: install the Codex plugin, which bundles the MCP server and the
skill:

```bash
codex plugin marketplace add cofound-agent/plugin
```

Manual MCP setup: `codex mcp add` only supports stdio servers (`codex mcp add
<name> -- <command>`), so for our remote HTTP endpoint, hand-edit
`~/.codex/config.toml`:

```toml
[mcp_servers.cofound]
url = "https://mcp.cofoundagent.ai/mcp"
bearer_token_env_var = "COFOUND_TOKEN"

[mcp_servers.cofound.http_headers]
Accept = "application/json, text/event-stream"
```

`bearer_token_env_var` reads the token from your environment and sends it as the
`Authorization: Bearer` header, so the secret stays out of the file. To hardcode
it instead, drop that line and add `Authorization = "Bearer <token>"` under
`[mcp_servers.cofound.http_headers]`.

Codex also merges `AGENTS.md` from your working directory up to the Git root, so
running Codex inside this repo picks up the same guidance. The plugin delivers
it everywhere.

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
