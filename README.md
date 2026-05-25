# Cofound Plugin

MCP-native co-founder directory. Your AI agent searches the directory, screens
inbound pitches, and drafts replies. You only see the conversations worth your
time.

- Website: https://cofound.dev
- Tool reference: [`docs/tools.md`](./docs/tools.md)
- Per-client setup: [`docs/setup.md`](./docs/setup.md)
- Agent instructions: [`SKILL.md`](./SKILL.md)

## What's in this repo

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json` + `marketplace.json` | Claude Code plugin install |
| `.cursor-plugin/plugin.json` + `marketplace.json` | Cursor plugin install |
| `skills/cofound/SKILL.md` | Auto-discovered by Claude Code and Cursor plugins |
| `SKILL.md` | Canonical agent instructions (paste into any system prompt) |
| `AGENTS.md` | Picked up by Codex CLI from repo root |
| `docs/tools.md` | Reference for all 12 MCP tools |
| `docs/setup.md` | Step-by-step setup for every supported client |

## Install

You need a Cofound MCP token first. Sign up at
[cofound.dev](https://cofound.dev), complete the attestation, then copy your
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

Install from the Cursor Marketplace (search "cofound"), or add the repo as a
plugin source in Cursor 2.5+ settings. The same `plugin.json` manifest is
recognized.

### Codex CLI

Codex has no bundle format; install the MCP server and let Codex pick up
`AGENTS.md` from this repo:

```bash
codex mcp add cofound \
  --url https://psmwvcglnckeqgyzlhth.supabase.co/functions/v1/mcp \
  --header "Authorization=Bearer $COFOUND_TOKEN" \
  --header "Accept=application/json, text/event-stream"
```

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
