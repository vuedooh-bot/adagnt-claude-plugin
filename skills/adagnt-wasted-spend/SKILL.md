---
name: adagnt-wasted-spend
description: Wasted-spend audit across all eight platforms — find budget going to non-converting terms, audiences, and placements; mine negative keywords from search terms; propose budget reallocation. Use for "where am I wasting money", cost-cutting passes, or monthly hygiene.
---

# AdAgnt Wasted-Spend Audit

Find money leaving the account without producing conversions, prove it with data, and stop it — with the user approving every cut. Target output: a dollar figure ("$X/month is going to Y for zero conversions") plus the fixes.

## Step 1 — Scope

Read `STRATEGY.md` for target ROAS/CPA (waste is defined against these) and check Performance History for previous audits — don't re-flag things the user consciously chose to keep. Default window: last 30–60 days; too short and low-volume keywords look unfairly bad.

## Step 2 — Run the platform audits

Run every audit for platforms with spend; each returns ranked waste findings:

- Google: `analyze_wasted_spend` (pass `target_roas` when the strategy defines one)
- Meta: `analyze_meta_wasted_spend`
- TikTok: `analyze_tiktok_wasted_spend`
- Amazon: no single audit tool — pull `amazon_get_sp_search_terms_report` and treat search terms with spend and zero attributed sales as the waste; negate with `amazon_create_sp_negative_keywords`
- AppLovin: `applovin_get_advertiser_report` against the campaign's `applovin_set_roas_target`

## Step 3 — Mine search terms for negatives (Google)

The single richest source of waste is queries that trigger ads but never convert:

1. Call `analyze_search_terms` with a matching `lookback_days` and a `min_clicks` floor (e.g. 5) so you judge terms with real data.
2. Sort candidates into:
   - **Irrelevant intent** — job seekers, DIY, free-seekers, wrong product entirely → negative, exact or phrase as appropriate
   - **Money drains** — relevant-looking terms with meaningful spend, zero conversions over the full window → negative or bid down, case by case
   - **Wrong-bucket terms** — converting terms landing in the wrong ad group → add as exact keywords where they belong (`add_keywords`) so they stop cross-matching
3. Propose the negative list grouped by theme with the spend each theme burned. Apply only after approval with `add_negative_keywords` (mistakes are reversible via `remove_negative_keywords`, but blocking a converting term costs real revenue — double-check anything ambiguous with the user).

## Step 4 — Audience and placement waste (Meta / TikTok)

- Meta: `analyze_meta_audiences` for saturated or overlapping audiences; `optimize_meta_placements` for placements that spend without converting; `detect_meta_creative_fatigue` when frequency is high and CTR is sliding.
- TikTok: `analyze_tiktok_geo_performance` for regions that drain budget; `detect_tiktok_creative_fatigue` for worn-out creative.

Also confirm the "waste" is real before cutting: `audit_conversion_tracking` — untracked conversions look identical to no conversions.

## Step 5 — Reallocate what you saved

Preview a rebalance with `optimize_budget_allocation` (Google; use `max_change_percentage` to keep moves conservative, `min_daily_budget` to protect small campaigns) and `optimize_meta_budget` / `optimize_tiktok_budget`. Present the before/after budget table.

## Step 6 — Present, apply, log

Deliver the audit as: total estimated monthly waste, findings ranked by dollar impact, and the proposed actions (negatives to add, placements/audiences to exclude, campaigns to pause via `pause_campaign` and platform variants, budgets to shift).

Apply only what the user approves — one call per write tool, verify with a read tool after, never auto-retry a failed write. Then append a dated entry to STRATEGY.md → Performance History: waste found, actions taken, expected monthly savings, and a note to re-check impact at the next review.
