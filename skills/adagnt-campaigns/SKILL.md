---
name: adagnt-campaigns
description: End-to-end campaign creation on the ad accounts a person connected — strategy-grounded keyword research, intent clustering, structured build-out, and every change filed as a proposal the account owner approves. Use when the user wants to launch or expand paid campaigns.
---

# AdAgnt Campaign Creation

Take a campaign from idea to live, grounded in the workspace strategy. The workflow has a research half (cheap, repeatable) and a build half (real money, so every change waits for the account owner's approval).

## Non-negotiable safety rules

- **Changes to an ad account wait for approval.** On a connected account a `create_*`, `add_*`, `update_*` or budget call files a proposal and returns `status: pending_approval`. Nothing changes until the account owner approves it on the card in the chat or in the AdAgnt app. You cannot approve your own proposal.
- **Say the plan in full first.** Name, budget, targeting, creative, and the status the campaign is created in, so the card holds no surprises. A campaign with its ad groups, keywords and ads goes in one `propose_plan`: one card, one approval.
- **Call each write tool exactly once.** If a write call errors or times out, STOP. Do not retry automatically: read `list_pending_actions` and the matching list tool (`list_campaigns`, `amazon_list_sp_campaigns`, `reddit_list_campaigns`, `x_ads_list_campaigns`) before deciding anything else.
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

After a change is approved, AdAgnt reads the platform back and the card reports what it found. Read the structure yourself as well (`get_campaign_structure`, `amazon_list_sp_campaigns`, `applovin_get_campaign`, or the platform's list tool) and give the user the created IDs.

**Report what the read observed, never what the write returned.** A write that reports failure has still sometimes succeeded, and the read is what tells the user which happened.

### Paused or live

Google and TikTok campaign tools create the campaign with its switch ENABLED once the proposal is approved; the other platforms create it PAUSED so its structure can be reviewed first. Say which in the plan, before the approval. To land a Google campaign paused, put `pause_campaign` in the same `propose_plan`, after the create, so one approval does both. Report the status in the platform's own words, and both facts: the switch, and what the platform has in effect. Never say a campaign is paused, live, running or spending from a write's result alone, and never say live or spending unless the platform reports it is delivering. Starting a paused campaign is its own change (`resume_campaign` and the platform equivalents) with its own approval.

### Hard limits on Google ad copy

`create_search_campaign` pins ad copy: **exactly 15 headlines, exactly 4 descriptions, and at least 5 keywords** per ad group. Fewer is rejected with `INVALID_ARGS`. Write the full set yourself rather than asking the user for fifteen headlines, and never promise them a smaller number — Google itself allows 3-15 headlines and 2-4 descriptions, so the constraint is ours and stricter than the platform's.

## Step 5 — Record and hand off

- Append a dated entry to STRATEGY.md → Performance History: what was launched, IDs, budget, hypothesis, and the date to review it (typically 7–14 days out).
- State each campaign's status in the platform's words, and give the one action that changes it — the platform's resume tool to start, its pause tool to stop; each is a proposal the account owner approves. `adagnt-performance` handles the review.

## Intelligence Layer

When two creative directions both look plausible, don't pick — test: `create_ab_test` runs variant A vs B inside the campaign until statistical significance, and `promote_ab_test_winner` concludes it (same confirm-before-write rule; both are writes). Check `get_ab_test_results` at the review date before judging creative by gut feel.
