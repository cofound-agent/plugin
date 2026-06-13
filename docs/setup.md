# Per-Client Setup

Cofound's MCP endpoint is HTTP Streamable:
`https://mcp.cofoundagent.ai/mcp`

## OAuth first, static token as fallback

**OAuth browser sign-in is the primary connect path.** For clients that support
MCP OAuth (Claude Code, Cursor, and Codex CLI), you sign in once in the browser, approve the
connection, and your agent is connected. The Claude
Code and Cursor sections below use the plugin install / one-click deeplink that
trigger this flow.

**The static bearer token is the fallback** for clients that do not support MCP
OAuth (today: ChatGPT, OpenClaw, Hermes, and other generic MCP
clients), and for any environment where the browser flow is unavailable. On that
path, every client needs the same two headers on every request:

- `Authorization: Bearer <token>`
- `Accept: application/json, text/event-stream`

The `Accept` header is required. The hosted endpoint returns `406 Not
Acceptable` on `initialize` if it's missing.

> **Tokens are show-once.** Copy the token from `/tokens` immediately after
> sign-up. If you lose it, regenerate; the previous token is revoked instantly,
> so update every client that used it.

## Claude Code

Install the plugin, then connect with a one-time OAuth browser sign-in:

```bash
/plugin marketplace add cofound-agent/plugin
/plugin install cofound
```

After installing, restart Claude Code (or run `/reload-plugins`) so the MCP
server loads, then run `/mcp` and authenticate **cofound** to sign in and approve
in the browser. The plugin manifest ships no `Authorization` header, so the
server's 401 triggers the browser OAuth flow instead of failing.

Fallback (static token): if OAuth is unavailable, `export
COFOUND_TOKEN="<your-token>"` before installing, or install without the plugin by
adding this to `.mcp.json` in your project or `~/.claude.json` for user-wide:

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

One-click: open [`cursor-deeplink.md`](./cursor-deeplink.md) and click "Add to
Cursor", then complete browser sign-in (OAuth). Once the plugin is published to
the Cursor Marketplace (Cursor 2.5+), you'll also be able to install it from
there.

Fallback (static token): if OAuth is unavailable, export `COFOUND_TOKEN` before
clicking the deeplink, or add the server manually:

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

Codex connects two ways: the plugin (bundles the MCP server and the skill) or a
direct `[mcp_servers.*]` entry (server only). Both use the same browser OAuth.

### Plugin (recommended)

Register the marketplace, then install and enable the plugin from Codex's plugin
browser:

```bash
codex plugin marketplace add cofound-agent/plugin
```

Open Codex, select **cofound** in the plugin browser, install it, and toggle it
on. The plugin declares `authentication: ON_INSTALL`, so Codex runs the OAuth
browser sign-in when you enable it, the same flow the official remote-MCP plugins
(cloudflare, etc.) use. The agent then has the cofound tools and the skill.

### Direct config (server only)

`codex mcp add cofound --url https://mcp.cofoundagent.ai/mcp` creates the entry,
but `codex mcp add` has no `--header` flag and the endpoint requires `Accept:
application/json, text/event-stream`, so add the block by hand:

```toml
[mcp_servers.cofound]
url = "https://mcp.cofoundagent.ai/mcp"

[mcp_servers.cofound.http_headers]
Accept = "application/json, text/event-stream"
```

```bash
codex mcp login cofound   # opens the browser for OAuth sign-in
```

`codex mcp login`, `codex mcp list`, and `codex mcp get` only operate on
`[mcp_servers.*]` entries, never on plugin-bundled servers, so pick one path (not
both with the name `cofound`).

Static-token fallback (if you prefer not to use OAuth): add
`bearer_token_env_var = "COFOUND_TOKEN"` under `[mcp_servers.cofound]` and export
that variable, or hardcode `Authorization = "Bearer <token>"` under
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
- **`401 Unauthorized`**: token is missing, malformed, or revoked. On OAuth
  clients (Claude Code, Cursor), a 401 triggers a fresh browser sign-in; only
  regenerate a static token at
  [cofoundagent.ai/tokens](https://cofoundagent.ai/tokens) if you are on the
  token fallback.
- **`forbidden` from `search_profiles`**: your own profile isn't ready. Check
  `details.missing_required_fields` and complete those via `update_my_profile`
  and `submit_raw_profile`.
- **Token security**: never paste your bearer token, auth header, raw profile
  text, message bodies, or private emails into Cursor prompts, logs,
  screenshots, or shared documents.
