---
name: adagnt-tiktok-ads
description: TikTok Ads through AdAgnt — video and carousel campaigns, ad groups, targeting, geo performance, creative-fatigue detection and asset upload. Use for any TikTok ads task. 29 tools.
---

# TikTok Ads

29 tools: 14 read, 15 write.

## Start here

`list_tiktok_campaigns`, `get_tiktok_campaign_details`,
`get_tiktok_campaign_performance`. Structure is campaign → ad group → ad;
`list_tiktok_ad_groups` and `list_tiktok_ads` drill down.

Everything needs an `advertiser_id`.

## Assets before campaigns

TikTok rejects work whose creative is not already uploaded and valid. Do this
first, every time:

- `validate_and_prepare_tiktok_assets`
- `upload_tiktok_images`
- `validate_video`

`discover_tiktok_assets` shows what is already available.

## Building

- `create_tiktok_video_campaign` — the default; TikTok is a video platform
- `create_tiktok_campaign` — the generic entry point for other formats

There is no carousel *campaign* tool. Carousels are built as a generic campaign
plus cards: `create_tiktok_campaign`, then `create_tiktok_carousel_card` per
card, then `add_tiktok_ad`.

Extend with `add_tiktok_ad_group` and `add_tiktok_ad`.

## Targeting

`search_tiktok_targeting`, `get_tiktok_audience_insights`. TikTok's interest
targeting is broader than Meta's; over-narrowing starves delivery.

## Creative fatigue is the TikTok-specific failure

`detect_tiktok_creative_fatigue` — TikTok creative decays faster than anywhere
else in the stack. When performance drops, check fatigue **before** touching
budget or targeting; the answer is usually a new video.

## Optimising

`analyze_tiktok_wasted_spend`, `analyze_tiktok_geo_performance`,
`optimize_tiktok_budget`, `explain_tiktok_anomaly`.

Geo performance is worth a look on TikTok specifically — delivery skews
regionally more than the other platforms, and `analyze_tiktok_geo_performance`
often finds a region carrying or dragging the whole campaign.

## Pausing

**TikTok campaigns are created ENABLED and start spending.** Call
`pause_tiktok_campaign` immediately after creating, in the same turn, unless the
user asked for it to go live — then read the status back before saying it is
paused.

When fixing a live account, pause at the level the problem lives:
`pause_tiktok_ad`, `pause_tiktok_ad_group`, `pause_tiktok_campaign`. Pausing a
whole campaign for one bad ad is a blunt instrument — say which you are
proposing and why.
