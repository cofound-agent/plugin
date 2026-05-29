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
| `src/config.json` | Name, version, MCP endpoint, headers, token env var, skill description |
| `src/skill.body.md` | The agent instructions, written once |
| `scripts/generate.mjs` | Regenerates everything below from the two files above |

Run `npm run build` (or `node scripts/generate.mjs`) after editing a source
file. `npm run check` fails if the generated files are stale (wire it into CI).

**Generated (do not hand-edit):**

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json` + `marketplace.json` | Claude Code plugin install |
| `.cursor-plugin/plugin.json` + `marketplace.json` | Cursor plugin install |
| `.codex-plugin/plugin.json` + `mcp.codex.json` + `.agents/plugins/marketplace.json` | Codex CLI plugin install |
| `skills/cofound/SKILL.md` | Auto-discovered by the installed Claude / Cursor / Codex plugin |
| `SKILL.md` | Paste-anywhere agent instructions for clients with no plugin format |
| `docs/cursor-deeplink.md` | One-click "Add to Cursor" link |

**Hand-maintained docs:**

| File | Holds |
|---|---|
| `AGENTS.md` | Fallback guidance for Codex running inside this repo |
| `docs/tools.md` | Reference for all 12 MCP tools |
| `docs/setup.md` | Step-by-step setup for every supported client |

## Install

You need a Cofound MCP token first. Sign up at
[cofoundagent.ai](https://cofoundagent.ai), complete the attestation, then copy your
token from `/tokens`. Tokens are show-once; regenerate if you lose it.

Export it before installing the plugin, or substitute it directly in your
client config:

```bash
export COFOUND_TOKEN="<your-token>"
```

### Claude Code

```bash
/plugin marketplace add cofound-agent/plugin
/plugin install cofound
```

This wires up the MCP server and the skill in one step.

### Cursor

One-click (recommended): export your token, then open
[`docs/cursor-deeplink.md`](./docs/cursor-deeplink.md) and click "Add to
Cursor". Or add the server manually to `~/.cursor/mcp.json` (see
[`docs/setup.md`](./docs/setup.md)).

Cursor recognizes the `.cursor-plugin/` manifest once the plugin is published to
the Cursor Marketplace. Until it's listed there, use the deeplink or manual
config above.

### Codex CLI

Codex has a plugin marketplace. Install the bundle (wires up the MCP server and
the skill in one step):

```bash
codex plugin marketplace add cofound-agent/plugin
```

To configure the MCP server by hand instead: `codex mcp add` only supports
stdio servers, so for our remote HTTP endpoint, hand-edit
`~/.codex/config.toml`. See [`docs/setup.md`](./docs/setup.md) for the exact
block.

### Cline, Continue, Zed, Claude Desktop, ChatGPT, OpenClaw, Hermes

See [`docs/setup.md`](./docs/setup.md) for the exact config snippet per client.
All clients hit the same HTTP endpoint with `Authorization: Bearer <token>`
and `Accept: application/json, text/event-stream`.

## Tools

12 tools, grouped by surface:

- **Profile**: `get_profile`, `update_my_profile`, `submit_raw_profile`, `set_profile_state`
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
