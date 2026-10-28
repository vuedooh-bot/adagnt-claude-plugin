---
name: adagnt-amazon-ads
description: Amazon Ads through AdAgnt — Sponsored Products, Brands and Display, DSP, keyword harvesting from search terms, budget rules, bidding and attribution. Profiles come first. Use for any Amazon ads task. 65 tools.
---

# Amazon Ads

65 tools: 31 read, 34 write. The largest surface AdAgnt has after Google.

## Profiles come first — always

**Every Amazon call needs a profile.** A profile pairs an advertiser account
with a single marketplace: a US profile cannot see UK campaigns.

1. `amazon_list_profiles`
2. `amazon_get_profile` for detail
3. Pass the profile explicitly from then on

Skipping this is the single most common Amazon failure. `amazon_list_ad_accounts`
and `amazon_list_portfolios` show the wider account structure.

## Three ad products, three build paths

**Sponsored Products** — the workhorse, keyword and product targeted:

`amazon_create_sp_campaign` → `amazon_create_sp_ad_group` →
`amazon_create_sp_product_ad` → `amazon_create_sp_keywords` /
`amazon_create_sp_targets`

Each step needs the ID from the one before.

**Sponsored Brands** — banner and brand presence:
`amazon_create_sb_campaign` → `amazon_create_sb_ad_group` →
`amazon_create_sb_ad` → `amazon_create_sb_keywords` / `amazon_create_sb_targets`.
`amazon_get_sb_landing_page_options` shows valid destinations.

**Sponsored Display** — retargeting and audiences:
`amazon_create_sd_campaign` → `amazon_create_sd_ad_group` →
`amazon_create_sd_product_ad` → `amazon_create_sd_targets`, creative via
`amazon_create_sd_creative`.

## Search terms are the goldmine

Amazon has **no single wasted-spend audit tool.** The route is:

1. `amazon_get_sp_search_terms_report`
2. Terms with spend and zero attributed sales are the waste
3. Negate them with `amazon_create_sp_negative_keywords` (or
   `amazon_create_sp_negative_targets`)

And the same report in reverse finds growth:
`amazon_get_keyword_harvest_suggestions` promotes converting search terms into
exact-match keywords. `amazon_get_negative_keyword_suggestions` does the
opposite automatically.

This harvest-and-negate loop is the core of Amazon account management. Run it
regularly rather than once.

## Reporting

`amazon_get_sp_campaigns_report`, `amazon_get_sp_targeting_report`,
`amazon_get_sb_campaigns_report`, `amazon_get_sd_campaigns_report`,
`amazon_get_brand_metrics_report`, and
`amazon_get_top_of_search_impression_share` — the last is the one that explains
why a well-bid campaign is not being seen.

`amazon_create_report_export` for bulk, `amazon_create_attribution_report` for
off-Amazon traffic.

## Bids and budgets

`amazon_set_bidding_strategy`, `amazon_set_bid_adjustments`,
`amazon_update_campaign_budget`, `amazon_create_budget_rule`.
`amazon_get_bid_recommendations` and `amazon_get_budget_recommendations` before
guessing. `amazon_get_account_budget_usage` shows pacing.

## Amazon's own recommendations

`amazon_list_recommendations` then `amazon_apply_recommendation`. **Applying is
a write that changes a live campaign** — propose it with the actual change
shown, never apply in bulk because Amazon suggested it.

## Amazon has no pause tool

Campaigns are created in an enabled state and start spending, and there is no
`pause_*` tool for Amazon. Use `amazon_update_sp_campaign` (or
`amazon_update_sb_campaign`) with `state` set to paused, immediately after
creating, unless the user asked for it to go live. Read it back before saying
it is paused.

## DSP

`amazon_list_dsp_advertisers`, `amazon_create_dsp_order`,
`amazon_create_dsp_line_item`, `amazon_create_dsp_creative`,
`amazon_get_dsp_report`. Programmatic display, separate from the Sponsored
products above.
