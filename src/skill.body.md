# Cofound MCP Agent Skill

Use the Cofound MCP tools as the user's agent, not as a browser replacement.
The public docs cover profile completion, search, messaging, and block tools.
Full tool reference: https://github.com/cofound-agent/plugin/blob/main/docs/tools.md

## Profile Readiness

- Complete the user's own profile before searching: call `get_profile` with
  `view = private`, inspect `missing_required_fields`, then use
  `update_my_profile` and `submit_raw_profile` as needed.
- For first-run setup, call `get_profile` with `{ "view": "private" }`, patch
  only user-approved missing fields via `update_my_profile`, submit approved
  founder-history text via `submit_raw_profile`, then re-check private profile
  state before searching.
- Do not call `search_profiles` until `normalization_status` is `ready` and
  `missing_required_fields` is empty. If search returns `forbidden` with
  `details.missing_required_fields`, treat those fields as the next completion
  checklist.
- Use `set_profile_state` only for `active` or `paused`. Account deletion is a
  website/account action, not an MCP state change.

## Search Strategy

- Map the user's intent to reciprocal candidate filters. If the user has an idea
  and wants a co-founder, search with `looking_for_includes` set to
  `["idea_to_join"]` and optionally `["peer_founder"]`. If the user is open to
  joining, search with `["co_founder_for_my_idea"]` and optionally
  `["peer_founder"]`. If the user wants peer founders, start with
  `["peer_founder"]` and broaden only when useful.
- Start strict with the user's approved commitment, location or timezone,
  remote preference, languages, reciprocal intent, and 1-2 core skill or
  industry filters. Do not filter on free-text fields.
- Treat co-founder search as a thin-market problem. If strict search returns no
  or very few candidates, explain which constraint is narrowing the market and
  run one deliberate broader search. Broaden skills or industries before
  geography/timezone. Never broaden privacy or safety constraints.
- Use staged-commitment matching. Before recommending a deep commitment,
  encourage a first exchange that tests one costly signal: customer or traction
  evidence, a technical plan, domain proof, distribution access, or a one-week
  trial sprint.
- For idea owners, prioritize complementary profiles and help the user signal
  why the idea is worth the candidate's time. Avoid generic-interest messages
  that lack a proof point or next step.
- For users open to joining, screen idea owners for traction, customer evidence,
  commitment, and genuinely complementary skills. Do not let the user drift into
  unpaid execution for an underspecified idea.
- For peer-founder matches where both sides have ideas, frame the first call as
  merge-or-compare: compare unfair insights, commitment, and market evidence
  before discussing equity or who joins whose company.

## Messages And Threads

- Before a first message, call `get_profile` with `view = public` for the
  candidate and check existing context with `list_threads`, `get_thread`, or
  `list_sent` when relevant.
- Respect contact caps. Prioritize first messages to high-fit profiles, avoid
  repeated follow-ups while awaiting a delivered reply, and stop immediately on
  `rate_limited` or `new_contact_daily_limit`; report the cap instead of
  retrying.
- Disclose agent drafting when composing messages for the human to approve.
- When triaging inboxes, prioritize first messages from new contacts, then
  messages with recent delivered replies. Use `get_thread` for message detail,
  and remember it may mark delivered counterparty messages as read.

## Privacy And Safety

- Never include raw profile fields, raw resume text, token secrets, or private
  `linkedin_url` values in outbound drafts.
- Do not deanonymize, enrich externally, guess identities, or reverse-engineer
  bucketed schools, companies, dates, or locations from public profiles.
- Use block tools when the user asks to hide, unblock, or review blocked
  profiles. Treat `not_found` and silent-block behavior as opaque; do not infer
  whether a profile is deleted, paused, blocked, or non-reciprocal.
