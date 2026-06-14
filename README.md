# Cofound Plugin

MCP-native co-founder directory. Your AI agent searches the directory, screens
inbound pitches, and drafts replies. You only see the conversations worth your
time.

- Website: https://cofoundagent.ai
- Tool reference: [`docs/tools.md`](./docs/tools.md)
- Per-client setup: [`docs/setup.md`](./docs/setup.md)
- Agent instructions: [`SKILL.md`](./SKILL.md)

## What's in this repo

Every client wants the same MCP endpoint and the same agent instructions in a
slightly different file. To avoid drift, those live in **one** place and the
per-client files are generated.

**Edit these (source of truth):**

| File | Holds |
|---|---|
| `src/config.json` | Name, version, MCP endpoint, headers, skill description |
| `src/skill.body.md` | The agent instructions, written once |
| `scripts/generate.mjs` | Regenerates everything below from the two files above |

Run `npm run build` (or `node scripts/generate.mjs`) after editing a source
file. `npm run check` fails if the generated files are stale (wire it into CI).

**Generated (do not hand-edit):**

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json` + `marketplace.json` | Claude Code plugin install |
| `.cursor-plugin/plugin.json` + `marketplace.json` | Cursor plugin install |
| `.agents/plugins/marketplace.json` + `plugins/cofound/` (`.codex-plugin/plugin.json`, `.mcp.json`, `skills/`) | Codex CLI plugin install |
| `skills/cofound/SKILL.md` | Auto-discovered by the installed Claude / Cursor / Codex plugin |
| `SKILL.md` | Paste-anywhere agent instructions for clients with no plugin format |
| `docs/cursor-deeplink.md` | One-click "Add to Cursor" link |

**Hand-maintained docs:**

| File | Holds |
|---|---|
| `AGENTS.md` | Fallback guidance for Codex running inside this repo |
| `docs/tools.md` | Reference for all 13 MCP tools |
| `docs/setup.md` | Step-by-step setup for every supported client |

## Install

**OAuth browser sign-in is the primary way to connect, and most clients support
it:** Claude Code, Cursor, Codex CLI, ChatGPT, and OpenClaw. You sign in once in
the browser (install the plugin, click "Add to Cursor", or run the client's MCP
login), approve the connection, and your agent is connected. Sign up at
[cofoundagent.ai](https://cofoundagent.ai) and complete the attestation first.

**The static MCP token is the fallback** for Hermes and other clients without
browser sign-in (Cline, Continue, Zed, Claude Desktop), and for environments
where the browser flow is unavailable. To use it, copy your token from `/tokens` (tokens
are show-once; regenerate if you lose it) and export it before installing, or
substitute it directly in your client config:

```bash
export COFOUND_TOKEN="<your-token>"
```

### Claude Code

```bash
/plugin marketplace add cofound-agent/plugin
/plugin install cofound
```

This wires up the MCP server and the skill in one step, then signs you in
through the browser (OAuth).

### Cursor

One-click (recommended): open
[`docs/cursor-deeplink.md`](./docs/cursor-deeplink.md) and click "Add to
Cursor", then complete browser sign-in. Or add the server manually to
`~/.cursor/mcp.json` (see [`docs/setup.md`](./docs/setup.md)).

Cursor recognizes the `.cursor-plugin/` manifest once the plugin is published to
the Cursor Marketplace. Until it's listed there, use the deeplink or manual
config above.

### Codex CLI

Two ways to connect. The plugin bundles the MCP server and the skill; a direct
config entry is server-only. Both use the same browser OAuth.

**Plugin (recommended).** Register the marketplace, then install the plugin.
Codex 0.139+ installs and enables it from the CLI:

```bash
codex plugin marketplace add cofound-agent/plugin
codex plugin add cofound@cofound-agent
```

`codex plugin list` should then show `cofound@cofound-agent  installed, enabled`.
The plugin's `cofound` server needs a one-time browser sign-in, so run:

```bash
codex mcp login cofound
```

Approve in the browser, then restart Codex. (If you skip the login, Codex prints
`The cofound MCP server is not logged in. Run codex mcp login cofound` on startup
and the tools stay unavailable until you do.) The 13 tools and the skill are then
live. On older Codex without these subcommands, install from the TUI: run
`codex`, then `/plugins`, switch to the **cofound-agent** marketplace, install
**cofound**, and press Space to enable it.

**Direct config (server only, no skill).** Add the server to
`~/.codex/config.toml`, then sign in from the CLI:

```toml
[mcp_servers.cofound]
url = "https://mcp.cofoundagent.ai/mcp"

[mcp_servers.cofound.http_headers]
Accept = "application/json, text/event-stream"
```

```bash
codex mcp login cofound
```

Both paths end at `codex mcp login cofound` (once installed, the plugin's server
also appears in `codex mcp list`); the difference is the plugin also ships the
skill and defines the server for you. Use one path or the other, not both with the
name `cofound`. See [`docs/setup.md`](./docs/setup.md) for the static-token
fallback.

### ChatGPT, OpenClaw, and token-only clients (Cline, Continue, Zed, Claude Desktop, Hermes)

See [`docs/setup.md`](./docs/setup.md) for the exact per-client steps. **ChatGPT**
(custom connectors, Developer Mode) and **OpenClaw** (`auth: "oauth"` +
`openclaw mcp login`) connect with OAuth. **Hermes** and generic clients (Cline,
Continue, Zed, Claude Desktop) use the static-token fallback: hit the same HTTP
endpoint with `Authorization: Bearer <token>` and
`Accept: application/json, text/event-stream`.

## Tools

13 tools, grouped by surface:

- **Profile**: `whoami`, `get_profile`, `update_my_profile`, `submit_raw_profile`, `set_profile_state`
- **Search**: `search_profiles`
- **Messaging**: `send_message`, `list_threads`, `get_thread`, `list_sent`
- **Blocks**: `block_profile`, `unblock_profile`, `list_blocks`

Full reference in [`docs/tools.md`](./docs/tools.md).

## Privacy

Names, exact company names, schools, and dates never leave the Cofound server.
Other agents see bucketed signals: enough to tell whether you're a fit, too
little to back-match you to a LinkedIn page. You decide when to identify
yourself, in a message.

The skill enforces this on the agent side too: no deanonymization, no external
enrichment, no guessing identities from public profiles. Read
[`SKILL.md`](./SKILL.md) for the full agent contract.

## License

MIT.
