---
name: adagnt-setup
description: First-run onboarding for AdAgnt. Connects the MCP server, walks the user through OAuth for each ad platform, verifies every connection, and seeds a STRATEGY.md so all later work is context-aware. Use in a fresh workspace or whenever connections need re-checking.
---

# AdAgnt Setup

Bring a new workspace from zero to fully connected. Work through the phases in order and do not skip verification — every later skill assumes the checks below have passed.

## Phase 1 — Confirm the MCP connection

1. Check whether AdAgnt tools (for example `get_connections_status`) are available in this session.
2. If they are not, the server is not registered yet. Point the user at the connector configs in `plugins/` for their client:
   - Claude Code: `plugins/claude-code/.mcp.json`
   - Cursor: `plugins/cursor/mcp.json`
   - Codex: `plugins/codex/config.toml`
   - ChatGPT: `plugins/chatgpt/README.md`
   The default local endpoint is `http://localhost:8734/mcp`.
3. Once tools appear, call `get_usage_status` to confirm the account is active and to learn the plan's write allowance. Tell the user what plan they are on and how many changes they have left this month; ad accounts and reads are unlimited on every plan, so there is no account limit to report.

## Phase 2 — Connect ad platforms

1. Call `get_connections_status` (no arguments) to see which of Google Ads, Meta Ads, TikTok Ads, Amazon Ads, AppLovin, Reddit Ads and X Ads are already linked.
2. Ask the user which platforms they actually advertise on. Do not push them to connect platforms they do not use.
3. For each missing platform, direct the user to the AdAgnt dashboard to complete OAuth in their browser. Never ask for passwords, tokens, or API keys in chat — authorization happens only through the platform's own consent screen.
4. After each authorization, re-run `get_connections_status` until the platform shows as connected.

## Phase 3 — Verify accounts

1. Call `list_connected_accounts` for each connected platform (pass the `platform` argument) and show the user the accounts found.
2. If more than one account exists per platform, ask which is primary and call `switch_primary_account` accordingly.
3. Sanity-check read access with one cheap read per platform: `list_campaigns` (Google), `list_meta_campaigns`, `list_tiktok_campaigns`. Empty results are fine; errors are not — surface any error verbatim and stop until resolved.

## Phase 4 — Seed the strategy document

1. Look for `STRATEGY.md` in the workspace root. If it exists, read it and confirm with the user that it is current.
2. If it does not exist, gather context before writing it:
   - Call `infer_business_profile` (Google) to have the server derive vertical, audience, and goals from account history, when a Google account is connected.
   - Call `get_business_profile` to read anything already saved server-side.
   - Ask the user for whatever is still missing: what they sell, who buys it, main competitors, target CPA or ROAS, monthly budget, brand voice.
3. Create `STRATEGY.md` with the six sections defined by the `adagnt-strategy` skill: Business Profile, Target Audience, Competitive Landscape, Campaign Strategy, Brand Voice, Performance History. Leave `Performance History` as an empty log with a dated "workspace initialized" entry.
4. Mirror the durable facts server-side with `save_business_profile` (vertical, size, audience, geography, goal, seasonality) so tools that read the profile behave consistently.

## Done criteria

- MCP tools respond and quota is known.
- Every platform the user advertises on shows connected in `get_connections_status`.
- `list_connected_accounts` returns the expected accounts and a primary is chosen.
- `STRATEGY.md` exists with all six sections filled in (Performance History may be just the init entry).

Report a short summary: platforms connected, accounts selected, plan/quota, and the path to `STRATEGY.md`.
