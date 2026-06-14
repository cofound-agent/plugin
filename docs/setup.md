# Per-Client Setup

Cofound's MCP endpoint is HTTP Streamable:
`https://mcp.cofoundagent.ai/mcp`

## OAuth first, static token as fallback

**OAuth browser sign-in is the primary connect path, and most clients support
it:** Claude Code, Cursor, Codex CLI, ChatGPT, and OpenClaw. You sign in once in
the browser, approve the connection, and your agent is connected (each client's
section below shows its sign-in step).

**The static bearer token is the fallback** for Hermes and other generic clients
without browser sign-in, and for any environment where the browser flow is
unavailable. On that
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

Register the marketplace, then install the plugin. Codex 0.139+ installs and
enables it from the CLI:

```bash
codex plugin marketplace add cofound-agent/plugin
codex plugin add cofound@cofound-agent
```

`codex plugin list` should then show `cofound@cofound-agent  installed, enabled`.
The plugin's `cofound` server needs a one-time browser sign-in:

```bash
codex mcp login cofound
```

Approve in the browser, then restart Codex. (If you skip it, Codex prints `The
cofound MCP server is not logged in. Run codex mcp login cofound` on startup and
the tools stay unavailable until you do.) The agent then has the cofound tools
and the skill. On older Codex without these subcommands, install from the TUI
instead: run `codex`, then `/plugins`, switch to the **cofound-agent**
marketplace, install **cofound**, and press Space to enable it.

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

Once the plugin is installed, its `cofound` server also appears in `codex mcp
list` and is authenticated the same way (`codex mcp login cofound`). Use either
the plugin or this config entry, not both with the name `cofound`.

Static-token fallback (if you prefer not to use OAuth): add
`bearer_token_env_var = "COFOUND_TOKEN"` under `[mcp_servers.cofound]` and export
that variable, or hardcode `Authorization = "Bearer <token>"` under
`[mcp_servers.cofound.http_headers]`.

Codex also merges `AGENTS.md` from your working directory up to the Git root, so
running Codex inside this repo picks up the same guidance. The plugin delivers
it everywhere.

## ChatGPT (custom connectors)

Requires a paid plan (Plus or Pro; Business/Enterprise/Edu where an admin enables
custom connectors). ChatGPT custom connectors authenticate with **OAuth**; the UI
has no field for a static token or an Accept header.

1. **Turn on Developer Mode:** Settings → Connectors (newer ChatGPT labels this
   "Apps & Connectors") → Advanced settings → enable **Developer mode (beta)**.
2. **Add the connector:** click **Create**, then enter:
   - Name: `Cofound`
   - MCP server URL: `https://mcp.cofoundagent.ai/mcp`
   - Authentication: `OAuth`
   - Check **"I trust this application"** (custom connectors are unverified beta).
3. **Authorize:** ChatGPT opens Cofound's browser sign-in; approve it and you are
   redirected back.
4. **Use it in a chat:** start a new chat, open **… More** in the tools/model
   selector → **Developer Mode** → select **Cofound**. Its tools are then
   available, and ChatGPT confirms each tool call (write actions need approval).
   Try "Use Cofound to run whoami."

## OpenClaw

OpenClaw supports OAuth. In `~/.openclaw/openclaw.json` (note the `mcp.servers`
nesting, not `mcpServers`; `transport` is required for remote servers), set
`auth: "oauth"`:

```json
{
  "mcp": {
    "servers": {
      "cofound": {
        "url": "https://mcp.cofoundagent.ai/mcp",
        "transport": "streamable-http",
        "auth": "oauth"
      }
    }
  }
}
```

Then run the browser sign-in (static `Authorization` headers are ignored while
`auth: "oauth"` is set):

```bash
openclaw mcp login cofound                # prints an authorization URL; approve it
openclaw mcp login cofound --code <code>  # paste the returned code to finish
```

Static-token fallback: omit `auth`, and add a `headers` block instead:

```json
{
  "mcp": {
    "servers": {
      "cofound": {
        "url": "https://mcp.cofoundagent.ai/mcp",
        "transport": "streamable-http",
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

`~/.hermes/config.yaml`:

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
  clients (Claude Code, Cursor, Codex, ChatGPT, OpenClaw), a 401 triggers a fresh
  browser sign-in; only regenerate a static token at
  [cofoundagent.ai/tokens](https://cofoundagent.ai/tokens) if you are on the
  token fallback.
- **`forbidden` from `search_profiles`**: your own profile isn't ready. Check
  `details.missing_required_fields` and complete those via `update_my_profile`
  and `submit_raw_profile`.
- **Token security**: never paste your bearer token, auth header, raw profile
  text, message bodies, or private emails into Cursor prompts, logs,
  screenshots, or shared documents.
