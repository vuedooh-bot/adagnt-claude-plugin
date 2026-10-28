---
name: adagnt-revenue
description: Set the revenue assumptions AdAgnt uses (provider label and average order value), then read per-campaign ROAS estimates with scale / keep / fix / kill verdicts. Revenue is modelled as conversions × average order value, never pulled from GA4, Shopify, Stripe or Klaviyo. Use when the user asks what their ads earn, wants revenue-based verdicts, or questions platform-reported conversion value.
---

# AdAgnt Revenue Estimates & ROAS Verdicts

What this skill does, and does not do. AdAgnt does not connect to GA4, Shopify, Stripe or Klaviyo and ingests no orders from them; no such connector exists. The revenue tools take two assumptions — which provider the user sells through (stored as a label) and an average order value — and model revenue per campaign as the platform's reported conversions × that order value. Every revenue figure in this skill is an estimate built on the ad platforms' own conversion counts. Every result carries a `revenue_basis` field saying so; repeat it to the user in plain words each time you show a number. Never call these figures real, measured, verified or provider-confirmed revenue, and never say a store or analytics account is "connected".

Why it still earns its place: conversion *value* is what each platform claims it drove, and it is often the weakest number in the report. Modelling revenue from conversion counts and one order value the user vouches for gives a consistent yardstick across platforms. The verdicts rank campaigns against each other; they are not an accounting of profit.

## Step 1 — Set the revenue assumptions

Check what is saved with `list_revenue_sources`. If nothing is set, or the user wants different assumptions, save them with `connect_revenue_source`:

- `source` — one of `ga4`, `shopify`, `stripe`, `klaviyo`: the provider the user sells through. It is kept as a label on the assumptions only; nothing is connected to it or pulled from it. Say that before you call.
- `property_id` — optional store or property identifier, stored next to the label for the user's reference.
- `average_order_value` — the number that drives every estimate (default 120). Ask for a realistic figure for the user's business rather than defaulting; a wrong order value scales every verdict with it.

Saving the assumptions is a write, but to AdAgnt's own records, not to any ad account or provider. Confirm the provider label and order value with the user first. The call backfills a 90-day modelled revenue baseline from every connected platform's campaign conversions and returns `revenue_basis` and a message saying the figures are modelled; pass that on. Verify with `list_revenue_sources` afterward.

One set of assumptions is enough. If the user saves several, the modelled events are joined from all of them, so make sure the order values describe different parts of the business rather than the same orders twice.

## Step 2 — Read the ROAS estimates

`get_true_roas` returns, per campaign, modelled revenue ÷ spend, ranked worst-to-best, each with a verdict. Thresholds: kill below 0.8, fix from 0.8, keep from 1.5, scale from 3.0. The tool name says "true"; the numbers are modelled, so present them as estimates. What the verdicts mean and what to do:

- **kill** — spend with no modelled revenue to show for it over the window, which means few or no reported conversions. Recommend pausing; the burden of proof is on the campaign now. Check one thing first: is the sales cycle longer than the date range? A 30-day window will condemn a 60-day B2B cycle unfairly.
- **fix** — conversions are coming in, but the ratio is bad at the user's order value. Something specific is usually broken: targeting, landing page, bid strategy or creative. Diagnose before touching budget — `adagnt-performance` has the anomaly tools.
- **keep** — earning its budget on the estimate. Leave it alone. Not every campaign needs an intervention this week.
- **scale** — high estimated ROAS with headroom. Propose a budget increase in steps (20–30% at a time), not a doubling — efficiency usually falls as spend rises.

Set `min_spend` so tiny campaigns don't pollute the ranking (default 10 is fine; raise it on large accounts). Use the same `date_range` the user's targets are stated in.

Present the ranking worst-first, exactly as returned — the user's biggest losses are the lede, not the wins.

## Step 3 — Explain the gap between modelled revenue and platform value

`get_revenue_attribution` returns, per campaign and platform, the platform-reported conversion value, the modelled revenue (conversions × order value), spend, both ROAS figures and the gap between them. When the user asks why the numbers disagree, the honest explanation:

- **Different inputs, same conversions.** Platform conversion value is the value each platform assigns to the conversions it claims; the modelled figure applies the user's one order value to the same conversion counts. A gap means the platform's per-conversion value differs from the user's order value. It does not mean one side has seen the orders.
- **Platforms over-claim across each other.** Each platform attributes any conversion it touched, so one order can be counted by Google *and* Meta. The modelled figure inherits this: it is built on each platform's conversions, so it does not de-duplicate across platforms either. Say so before summing anything.
- **Under-reporting happens too.** Blocked tracking, iOS privacy limits and offline sales make platforms miss conversions, and the model misses them for the same reason, because it starts from the same counts.

The rule of thumb to give the user: **platform-reported value for optimizing within a platform, the modelled estimate for a consistent comparison across platforms, and neither for a profit claim.** A campaign is never "actually profitable" on either figure alone; that needs order data AdAgnt does not have.

## Step 4 — Act on it

Verdicts are recommendations, not actions. Any pause, budget change or reallocation that follows goes through the normal write-tool discipline: exact payload shown, explicit yes, one call, verify with a read tool. For cross-platform budget moves based on the ROAS estimates, `optimize_cross_platform_budget` files a plan to the approval queue by default — keep it that way. Record what was decided, and the order value it rested on, with `update_strategy` (`append_learning`) so next month's review starts from this one's conclusions.
