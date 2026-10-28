---
name: adagnt-snapchat-ads
description: Snapchat Ads through AdAgnt — Snap Ads, Story Ads and Collection Ads as campaigns, ad squads and creatives, audiences and pixels, interest/geo/device targeting and stats with breakdowns. Use for any Snapchat ads task. 34 tools.
---

# Snapchat Ads

34 tools: 18 read, 16 write. Snapchat's hierarchy is organization → ad
account → campaign → **ad squad** (its ad group) → ad, with creatives and
media kept as account-level libraries that ads point at.

## Know this before you start

- **Money is in micros** on the wire; AdAgnt converts at the edge, so you pass
  and read ordinary currency amounts. Say "$25/day", never "25000000".
- **Everything is created PAUSED.** `snapchat_create_campaign`,
  `snapchat_create_ad_squad` and `snapchat_create_ad` never spend on creation;
  a separate confirmed `snapchat_resume_*` turns delivery on.
- **Targeting on an ad squad is replaced, not merged.** Any targeting field
  passed to `snapchat_update_ad_squad` overwrites that whole field. Read the
  squad with `snapchat_get_ad_squad` first and send the complete new set.
- **Creative changes re-enter Snapchat review.** Swapping a creative with
  `snapchat_update_ad` or creating a new ad means the ad serves only after
  Snapchat approves it; say so when you propose it.
- Until the user links a Snapchat account, every tool runs on the sandbox
  (an ad account with campaigns, squads, ads and 90 days of stats). If the
  account is not linked and the user wants real data, point them to
  `get_connections_status`.

## Start here

`snapchat_list_ad_accounts` (or `snapchat_list_organizations` when the user
has several businesses), then `snapchat_list_campaigns`,
`snapchat_list_ad_squads`, `snapchat_list_ads`. `snapchat_get_ad_account`
shows currency, timezone and funding; `snapchat_list_funding_sources` when a
campaign cannot spend.

## Reading performance

`snapchat_get_stats` works at account, campaign, ad squad or ad level with
`granularity` TOTAL, DAY or HOUR and breakdowns by ad squad or ad. The fields
that matter: impressions, swipes (Snapchat's click), spend, video views,
conversions. Swipe-up rate and cost per swipe are the first diagnostics;
`snapchat_analyze_wasted_spend` finds squads that spent above a floor with
zero conversions and is read-only.

## Building

1. `snapchat_search_targeting` for interests (SCC categories), geo, devices
   and OS — use the ids it returns, never guess them.
2. `snapchat_create_campaign` (objective, daily or lifetime spend cap, start).
3. `snapchat_create_ad_squad` — type (SNAP_ADS, STORY_ADS, COLLECTION…),
   optimization goal, bid strategy and bid, budget, placement, targeting. A
   PIXEL_* goal needs a pixel: `snapchat_list_pixels`.
4. Media and creative: `snapchat_upload_media` from an https URL →
   `snapchat_create_creative` (headline max 34 characters, brand name, call
   to action, top snap media) → `snapchat_create_ad`.
5. Propose the whole build as one plan and get one approval; then
   `snapchat_resume_campaign` / `snapchat_resume_ad_squad` only when the user
   says go.

Audiences: `snapchat_list_audiences` to reuse, `snapchat_create_audience` for
a customer-list segment (filled in Ads Manager) or a lookalike of an existing
segment.

## Changing spend and status

`snapchat_set_budget` for a campaign's spend cap or a squad's budget;
`snapchat_update_campaign` / `snapchat_update_ad_squad` for schedule, bid and
name; `snapchat_pause_*` and `snapchat_resume_*` for status. Every write
returns the object before and after the change — quote both when you report.
