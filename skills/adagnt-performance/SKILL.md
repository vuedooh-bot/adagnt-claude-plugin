---
name: adagnt-performance
description: Cross-platform performance review — pull metrics from Google, Meta, TikTok, Amazon, and AppLovin, compare ROAS/CPA against targets, explain anomalies, and turn findings into ranked recommendations. Use for weekly reviews, "how are my ads doing", or investigating a metric change.
---

# AdAgnt Performance Review

Answer two questions with evidence: *is spend producing what the strategy targets?* and *what single change would improve results most?* Read-only until the user approves an action.

## Step 1 — Establish the yardstick

Read `STRATEGY.md` for target CPA/ROAS, monthly budget, and prior review notes in Performance History. A number is only good or bad relative to these targets. If no targets exist, ask for them (or agree on a provisional one) before judging anything.

Pick the window with the user: default to the last 30 days with the prior 30 as comparison. Avoid windows under 7 days — daily noise masquerades as trend.

## Step 2 — Pull the data

Query every connected platform that has spend:

- Google: `get_campaign_performance` (use `lookback_days` or `start_date`/`end_date`)
- Meta: `get_meta_campaign_performance`; drill into ads with `analyze_meta_ad_performance`
- TikTok: `get_tiktok_campaign_performance`; ad-level via `get_tiktok_ad_performance`
- Amazon: `amazon_get_sp_campaigns_report`; search terms via `amazon_get_sp_search_terms_report`, Sponsored Brands via `amazon_get_sb_campaigns_report`
- AppLovin: `applovin_get_advertiser_report`; cohort return via `applovin_get_roas_cohort_report`, creative-level via `applovin_get_creative_performance`

Add context where it helps: `get_benchmark_context` (Google vertical benchmarks), `get_campaign_targeting` and platform equivalents when targeting might explain results.

## Step 3 — Normalize and compare

Build one table across platforms: spend, impressions, clicks, CTR, CPC, conversions, CPA, revenue, ROAS — per campaign, with a platform subtotal and a grand total. Then flag:

- Campaigns above/below target CPA or ROAS (sorted by spend, so the biggest problems surface first)
- Spend concentration: does the top campaign deserve its share?
- Trend vs the prior window: what moved more than ~20%?
- Cross-platform efficiency: cost per conversion by platform — but note attribution differences (platform-reported conversions overlap; don't sum them as truth)

## Step 4 — Explain the anomalies

For any metric that moved sharply, get a causal read before recommending anything:

- Google: `explain_performance_anomaly` (pass `metric`, `period_start`, `period_end`)
- Meta: `explain_meta_anomaly` · TikTok: `explain_tiktok_anomaly`

Check creative fatigue when CTR decays with stable targeting: `detect_meta_creative_fatigue`, `detect_tiktok_creative_fatigue`. Verify tracking before blaming performance: a conversion cliff is often a broken tag — `audit_conversion_tracking`.

## Step 5 — Recommend, ranked by expected impact

Present at most five recommendations, each with: the evidence, the action, the tool that executes it, and the expected effect. Typical shapes:

1. **Reallocate budget** toward efficient campaigns — preview with `optimize_budget_allocation` (Google), `optimize_meta_budget`, `optimize_tiktok_budget`
2. **Pause** a chronic underperformer — `pause_campaign` and platform variants
3. **Refresh creative** where fatigue is detected — hand off to `adagnt-ad-copy`
4. **Fix bidding** — `update_bid_strategy` when the strategy fights the objective
5. **Cut waste** — if waste is a theme, run the full `adagnt-wasted-spend` audit instead of patching here

Every one of these except the preview calls writes to a live account: confirm explicitly, execute once, verify with a read tool, never auto-retry.

## Step 6 — Write back what you learned

Append a dated review entry to STRATEGY.md → Performance History: window, headline numbers vs target, anomalies explained, actions taken (with IDs), and the next review date. Optionally set up continuous watching: `create_monitor` / `test_monitor` for metric alerts, `schedule_brief` for recurring reports, `generate_report_now` for a one-off.

## Intelligence Layer

Platform-reported ROAS is the platform's claim about its own conversion value. If revenue assumptions are saved (`list_revenue_sources`), anchor Step 3 on `get_true_roas` — modelled revenue (conversions × the saved average order value) ÷ spend with a scale/keep/fix/kill verdict per campaign — and use `get_revenue_attribution` to explain gaps between reported conversion value and that estimate. Both rest on the platforms' own conversion counts; AdAgnt pulls no orders from any store or analytics provider, so present the figures as estimates. The `adagnt-revenue` skill covers the full workflow.
