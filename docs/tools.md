# Tool Reference

Cofound's MCP endpoint exposes 12 tools, grouped into four surfaces: profile,
search, messaging, and blocks. This page is the user-facing reference. The
protocol-level source of truth is the tool descriptions returned by the MCP
server's `tools/list` response; the agent contract for how to chain these
tools lives in [`../SKILL.md`](../SKILL.md).

## Profile

### `get_profile`

Read a private owner profile or an opaque public profile projection. Public
reads hide non-visible targets as `not_found`.

- Pass `{ "view": "private" }` to inspect your own profile.
- Pass `{ "view": "public", "profile_id": "<id>" }` before sending a first
  message to a candidate.
- The public projection includes the candidate's `idea` outline (when they have
  one) and bucketed `history`, the strongest signals for judging fit. Passing a
  `fields` array returns only those fields, so omit it, or include `idea` and
  `history`, to avoid scoping out the idea.

A private read returns `normalization_status` and `missing_required_fields`.
Both must be `ready` and empty respectively before `search_profiles` will
succeed.

### `update_my_profile`

Patch only writable profile, idea-slot-0, and location fields. Unknown,
read-only, and invalid fields are rejected.

Use this to fill `missing_required_fields` reported by `get_profile` or
`search_profiles`. Only patch fields the user has explicitly approved.

### `submit_raw_profile`

Submit freeform profile text through the same synchronous normalizer used by
the website.

Accepts text only. PDFs must be extracted to text on the website first; the
MCP path does not accept binary uploads. Use for resume bodies, LinkedIn
exports, or founder-history paragraphs that the normalizer can parse into
structured fields.

### `set_profile_state`

Set the caller profile state to `active` or `paused`. Account deletion is a
website action, not an MCP state change.

## Search

### `search_profiles`

Search reciprocal, ready, non-blocked public profiles using only structured
filters. Returns an opaque public projection per match: bucketed signals, no
raw identifying fields.

Each result is `{ profile, match }`. `profile` is the full public projection,
including the candidate's `idea` outline when they have one; `match` carries
`complementarity_score` (higher means stronger mutual fit), `distance_km`, and
`timezone_diff_hours`. Results are ordered by `complementarity_score`, so the
strongest fits come first. A `fields` array scopes only the `profile` part, so
include `idea` to keep the strongest fit signal.

Filtering guidance (see SKILL for the full strategy):

- Use `looking_for_includes` to set reciprocal intent: `idea_to_join`,
  `co_founder_for_my_idea`, `peer_founder`.
- Filter on `commitment_any_of`, `open_to_remote`, `languages_includes`,
  `industries_any_of`, `skills_any_of`, and location/timezone fields.
- Never filter on free-text fields.
- If results are empty, broaden skills or industries before geography or
  timezone. Never broaden privacy or safety constraints.

If your own profile is not ready, the server returns `forbidden` with
`details.missing_required_fields`. Treat those as the next
`update_my_profile` checklist.

## Messaging

### `send_message`

Send a message while preserving silent-block opacity and contact caps.

Contact caps:

- 10 first-messages-to-new-contacts per rolling 24 hours.
- Replies inside existing threads are uncapped.
- On `rate_limited` or `new_contact_daily_limit`, stop. Do not retry. Report
  the cap to the user.

Always disclose agent drafting when composing a message for the human to
approve.

### `list_threads`

List message thread metadata without marking messages read. Use as the entry
point for triage.

### `get_thread`

Read one thread and mark only delivered counterparty messages as read. Has
side effects; use after `list_threads` has identified the thread you want to
open.

### `list_sent`

List authored messages without side effects or exposing silent-block delivery
state. Use to inspect what the user (or agent) has already sent without
revealing whether a recipient silently blocked the thread.

## Blocks

### `block_profile`

Idempotently block a visible profile. Hidden targets return opaque
`not_found` so blocking does not leak target existence.

### `unblock_profile`

Idempotently unblock a profile.

### `list_blocks`

List currently visible profiles blocked by the caller, including newly
blocked targets.

## Error contract

Common error codes you'll see in tool responses:

| Code | Meaning | Recovery |
|---|---|---|
| `unauthorized` | Token missing, malformed, or revoked | On OAuth clients (Claude Code, Cursor), a 401 triggers a fresh browser sign-in; only regenerate a static token at `/tokens` if you are on the token fallback |
| `forbidden` | Profile not ready, or operation not allowed | Inspect `details.missing_required_fields` |
| `rate_limited` | Per-token or global rate cap hit | Stop, report to user, retry later |
| `new_contact_daily_limit` | 10 new-contact-messages-per-24h cap hit | Stop, switch to existing-thread replies |
| `not_found` | Target hidden, deleted, blocked, paused, or non-existent | Treat as opaque; do not infer the cause |
| `validation_error` | Input failed schema validation | Inspect `details` for field-level errors |

## Privacy invariants

- Names, exact company names, schools, and dates never appear in any public
  projection.
- `linkedin_url` is private. Never include it in outbound drafts.
- Treat `not_found` as opaque. Do not infer whether the target is deleted,
  paused, blocked, or non-reciprocal.
- Do not deanonymize, externally enrich, or reverse-engineer bucketed fields
  from public projections.

The agent contract in [`../SKILL.md`](../SKILL.md) elaborates on each of these.
