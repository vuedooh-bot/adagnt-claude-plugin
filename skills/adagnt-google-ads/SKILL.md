---
name: adagnt-google-ads
description: Google Ads through AdAgnt — Search, Performance Max, Demand Gen and YouTube campaigns, keyword and search-term work, extensions, bidding and wasted-spend analysis. Use for any Google Ads task. 50 tools.
---

# Google Ads

50 tools: 19 read, 31 write. The deepest surface AdAgnt has.

## Start here

`list_campaigns` for the account, `get_campaign_structure` for one campaign's
shape, `get_campaign_performance` for metrics. All read-only.

`select_google_campaign_type` helps choose a campaign type when the user is
unsure — cheaper than building the wrong one.

## Building a campaign

Research first: **`research_keywords`** returns real monthly volumes,
competition and top-of-page bid ranges. Never invent keywords.

| Type | Tool | Follow-ups |
|---|---|---|
| Search | `create_search_campaign` | `add_keywords`, `add_negative_keywords`, `create_ad` |
| Performance Max | `create_pmax_campaign` | `add_pmax_search_themes`, `add_pmax_audience_signal` |
| Demand Gen | `create_demandgen_campaign` | `add_demandgen_ad_group` |
| YouTube | `create_youtube_campaign` | `validate_video` first |

### The limit that rejects calls

`create_search_campaign` requires **exactly 15 headlines, exactly 4
descriptions, and at least 5 keywords** per ad group. Fewer returns
`INVALID_ARGS` and nothing is created. Write the full set yourself; do not ask
the user for fifteen headlines.

(`create_ad` and `update_ad_headlines` are looser: 3–15 headlines.)

## Extensions

`add_sitelinks`, `add_callout_extensions`, `add_structured_snippets`;
`list_campaign_extensions` to see what exists. Cheap wins on click-through that
most accounts neglect.

## Optimising

- `analyze_search_terms` — what people actually typed. The richest source of
  negative keywords.
- `analyze_wasted_spend` — where the spend went: wasted (proven), too early to call, working or not judged, for every Google account on the connection
- `optimize_budget_allocation` — proposals, never applied silently
- `explain_performance_anomaly` — why a metric moved
- `get_benchmark_context` — how the account compares to its industry

Bidding: `update_bid_strategy`. Keywords: `update_keyword`, `remove_keywords`.

## Business context

`infer_business_profile` from a website, `save_business_profile`, and
`get_business_profile` to reuse it. Grounding campaigns in the real business
beats generic copy.

## Writes

Every create/update/pause/remove here changes money. On a connected account
each one files a proposal the account owner approves; follow `adagnt-agent`:
say every field, file it once, report the read-back.

**A Google campaign is created with its switch ENABLED** once the proposal is
approved. Say so in the plan, before the approval. To land it paused, put
`pause_campaign` in the same `propose_plan`, after the create, so one approval
does both. Read the status back with `list_campaigns` or
`get_campaign_structure` and report Google's own words: ENABLED is the switch;
`primary_status` says what Google has in effect.
