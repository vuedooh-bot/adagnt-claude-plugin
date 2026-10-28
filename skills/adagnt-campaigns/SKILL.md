---
name: adagnt-campaigns
description: End-to-end campaign creation across Google, Meta, TikTok, Amazon, and AppLovin — strategy-grounded keyword research, intent clustering, structured build-out, and a hard confirmation gate before any tool that spends money. Use when the user wants to launch or expand paid campaigns.
---

# AdAgnt Campaign Creation

Take a campaign from idea to live, grounded in the workspace strategy. The workflow has a research half (cheap, repeatable) and a build half (spends real money, gated).

## Non-negotiable safety rules

- **Write tools cost real money.** Any `create_*`, `add_*`, `update_*`, or budget-touching call changes a live ad account.
- **Confirm before every write.** Show the exact payload — name, budget, targeting, creative — and get an explicit "yes" from the user before calling.
- **Call each write tool exactly once.** If a write call errors or times out, STOP. Do not retry automatically: the campaign may have been created despite the error. Verify with the matching read tool (`list_campaigns`, `list_meta_campaigns`, `list_tiktok_campaigns`) before deciding anything else.
- Never invent budgets. If the user has not given one, ask; check STRATEGY.md for a monthly budget to derive a daily figure from.

## Step 1 — Load context

Read `STRATEGY.md` (see the `adagnt-strategy` skill for its shape). Pull out: business description, audience, goals (target CPA/ROAS), budget guidance, brand voice, and any lessons in Performance History. If STRATEGY.md is missing, run `adagnt-setup` first.

Confirm the platform and objective with the user, and check the right account is connected via `get_connections_status`.

## Step 2 — Research

For search-driven campaigns (Google):

1. Call `research_keywords` with `business_description` and `website_url` from STRATEGY.md plus any `seed_keywords` the user gave. Include `target_location` when the business is geo-bound.
2. Cluster the results by intent, not by topic:
   - **Transactional** — buy/pricing/demo terms → exact/phrase match, highest bids
   - **Commercial investigation** — "best", "vs", "alternatives" → phrase match, mid bids
   - **Informational** — how/what/why → usually excluded or a separate low-bid group
   - **Branded** — the user's brand and its misspellings → own ad group
   - **Competitor** — rival brand names → only with the user's explicit sign-off
3. Each cluster becomes one ad group with tightly related keywords (roughly 10–20 per group). Propose starting negative keywords from the obviously irrelevant terms (jobs, free, DIY — whatever does not fit the business).

For audience-driven campaigns (Meta/TikTok), research targeting instead: `search_meta_targeting` / `browse_meta_targeting`, `search_tiktok_targeting`. Cross-check with `get_meta_audience_insights` or `get_tiktok_audience_insights` where useful.

If unsure which campaign type fits the objective, use `select_google_campaign_type` or `select_meta_campaign_type`.

## Step 3 — Propose the build

Present a single, complete plan before touching a write tool:

- Campaign name (convention: `{platform}-{objective}-{audience}-{yyyymm}`)
- Objective, budget (daily and, if used, lifetime), bidding strategy
- Ad group / ad set structure with keywords or targeting per group
- Creative: headlines, descriptions, primary text, images/video (per the `adagnt-ad-copy` skill; validate assets first with `validate_and_prepare_assets` or the platform-specific variants)
- Negative keywords (Google) or audience exclusions
- Landing page URL(s)

Then ask: proceed, adjust, or abort.

## Step 4 — Build (one call per write, verify after)

On explicit approval only:

- **Google Search**: `create_search_campaign` with `campaign_name`, `budget_daily`, `ad_groups` (keywords + creative per group), `negative_keywords`, `bidding_strategy`, `target_locations`. Performance Max: `create_pmax_campaign`, then `add_pmax_search_themes` / `add_pmax_audience_signal`. Demand Gen: `create_demandgen_campaign`. YouTube: `create_youtube_campaign`.
- **Meta**: `create_meta_image_campaign`, `create_meta_video_campaign`, or `create_meta_carousel_campaign` — pass `ad_account_id`, `facebook_page_id`, budget, targeting, creative fields. Set `special_ad_categories` when the vertical requires it (housing, credit, employment, politics). Extend with `add_meta_ad_set` / `add_meta_ad`.
- **TikTok**: `create_tiktok_campaign` or `create_tiktok_video_campaign` with `advertiser_id`, budget, targeting; upload assets first via `upload_tiktok_images` / `validate_video`. Extend with `add_tiktok_ad_group` / `add_tiktok_ad`.
- **Amazon**: `amazon_create_sp_campaign` (Sponsored Products) with a profile from `amazon_list_profiles`, then `amazon_create_sp_ad_group`, `amazon_create_sp_product_ad`, `amazon_create_sp_keywords`. Sponsored Brands: `amazon_create_sb_campaign`. Sponsored Display: `amazon_create_sd_campaign`.
- **AppLovin**: `applovin_create_campaign` (or `applovin_create_web_campaign`), then attach creative with `applovin_create_creative_set` / `applovin_attach_creative_set`, and set spend with `applovin_set_global_budget` and `applovin_set_roas_target`.

After each successful write, verify with the corresponding read tool (`get_campaign_structure`, `get_meta_campaign_details`, `get_tiktok_campaign_details`, `amazon_list_sp_campaigns`, `applovin_get_campaign`) and echo the created IDs to the user.

**Report what the read observed, never what the write returned.** A write that reports failure has still sometimes succeeded, and the independent read is what tells the user which happened.

### Land it paused

**Google, Meta, TikTok and Amazon campaigns are created ENABLED and start spending immediately.** AppLovin takes `status` on create. Unless the user asked for it to go live, pause it in the same turn as you create it — `pause_campaign`, `pause_meta_campaign`, `pause_tiktok_campaign`, or for Amazon `amazon_update_sp_campaign` with `state` (Amazon has no pause tool). Then read the status back before telling the user it is paused. A campaign the user believes is paused and is not is the worst outcome this workflow can produce.

### Hard limits on Google ad copy

`create_search_campaign` pins ad copy: **exactly 15 headlines, exactly 4 descriptions, and at least 5 keywords** per ad group. Fewer is rejected with `INVALID_ARGS`. Write the full set yourself rather than asking the user for fifteen headlines, and never promise them a smaller number — Google itself allows 3-15 headlines and 2-4 descriptions, so the constraint is ours and stricter than the platform's.

## Step 5 — Record and hand off

- Append a dated entry to STRATEGY.md → Performance History: what was launched, IDs, budget, hypothesis, and the date to review it (typically 7–14 days out).
- State clearly whether each campaign is live or paused, and give the one action that flips it — `resume_campaign` to start, `pause_campaign` / `pause_meta_campaign` / `pause_tiktok_campaign` to stop. `adagnt-performance` handles the review.

## Intelligence Layer

When two creative directions both look plausible, don't pick — test: `create_ab_test` runs variant A vs B inside the campaign until statistical significance, and `promote_ab_test_winner` concludes it (same confirm-before-write rule; both are writes). Check `get_ab_test_results` at the review date before judging creative by gut feel.
