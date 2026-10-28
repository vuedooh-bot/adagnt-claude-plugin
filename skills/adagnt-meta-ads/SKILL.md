---
name: adagnt-meta-ads
description: Meta Ads through AdAgnt — image, video, carousel and DCO campaigns on Facebook and Instagram, audiences, pixels, lead forms, placements and creative-fatigue detection. Use for any Meta or Instagram ads task. 36 tools.
---

# Meta Ads

36 tools: 21 read, 15 write. Read-heavy by design — Meta rewards understanding
the audience before spending.

## Start here

`list_meta_campaigns`, `get_meta_campaign_details`,
`get_meta_campaign_performance`. `list_meta_ad_sets` and `list_meta_ads` drill
down; Meta's hierarchy is campaign → ad set → ad, and budget usually lives at
the ad set.

`discover_meta_assets` finds what the account already has —
`list_meta_pixels`, `list_meta_instagram_accounts`,
`list_meta_custom_audiences`.

## Building

`select_meta_campaign_type` first if the format is unsettled, then:

- `create_meta_image_campaign`
- `create_meta_video_campaign`
- `create_meta_carousel_campaign`
- `create_meta_dco_ad` — dynamic creative, lets Meta assemble combinations

All need `ad_account_id` and a `facebook_page_id`. Validate creative first with
`validate_and_prepare_meta_assets`.

**Set `special_ad_categories`** when the vertical requires it — housing,
credit, employment, social issues. Getting this wrong is a policy violation,
not a performance problem.

Extend with `add_meta_ad_set` and `add_meta_ad`. `duplicate_meta_campaign`
clones a winner rather than rebuilding it.

## Audiences and targeting

`browse_meta_targeting` and `search_meta_targeting` to explore;
`analyze_meta_audiences` and `get_meta_audience_insights` to judge. Overlapping
audiences bid against each other — check before adding another.

## Creative fatigue is the Meta-specific failure

`detect_meta_creative_fatigue` finds ads whose performance is decaying from
repetition rather than from targeting. On Meta the fix is usually new creative,
not a new audience or a higher bid. Check fatigue before recommending a budget
change.

## Leads

`list_meta_lead_forms`, `get_meta_lead_form_submissions`. Lead volume without
lead quality is a vanity metric — say so when the data allows.

## Optimising

`analyze_meta_wasted_spend`, `optimize_meta_budget`, `optimize_meta_placements`,
`explain_meta_anomaly`. All produce proposals.

## Writes

**Meta campaigns are created ENABLED and start spending.** Call
`pause_meta_campaign` immediately after creating, in the same turn, unless the
user asked for it to go live — then read the status back before saying it is
paused. Otherwise follow `adagnt-agent`.
