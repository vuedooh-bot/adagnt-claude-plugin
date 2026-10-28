---
name: adagnt-applovin-ads
description: AppLovin through AdAgnt — mobile user-acquisition campaigns, creative sets, geo/OS/device targeting, budgets and ROAS targets, plus cohort and retention reporting. Use for any AppLovin task. 36 tools.
---

# AppLovin

36 tools: 12 read, 24 write. Mobile user acquisition, which behaves differently
from the search and social platforms.

## Know this before you start

**On a live AppLovin account, every write returns `NOT_SUPPORTED`** — campaign
creation and updates, creative-set creation and updates, asset creation and
updates. So do three reads: creative-set listing, asset listing, and targeting
search.

What *is* live: campaign listing and detail, account settings, and the whole
reporting surface (advertiser, ROAS cohort, retention, creative performance).

In sandbox all 36 tools work normally, so this only bites on real accounts.

This is a deliberate driver limitation — AdAgnt does not perform AppLovin writes
it has not validated against a real key, because a wrong write body
misconfigures a live campaign. It is not a bug and not a failed call. Say
plainly that AppLovin write support is not available yet rather than implying
something went wrong or retrying.

## Start here

`applovin_list_accounts`, then `applovin_list_campaigns` and
`applovin_get_campaign`. `applovin_get_account_settings` for account-level
configuration.

## Building

`applovin_create_campaign` for app install, `applovin_create_web_campaign` for
web. Then configure — AppLovin splits configuration across many small setters
rather than one large create call:

| Concern | Tools |
|---|---|
| Objective | `applovin_set_campaign_objective`, `applovin_set_optimization_goal` |
| Platform | `applovin_set_campaign_platform`, `applovin_set_os_targeting`, `applovin_set_device_targeting` |
| Geography | `applovin_set_geo_targeting`, `applovin_set_country_budgets` |
| Spend | `applovin_set_global_budget`, `applovin_set_roas_target` |
| Schedule | `applovin_set_campaign_schedule` |
| Status | `applovin_set_campaign_status` |

`applovin_create_campaign` takes `status` directly (`ACTIVE` or `PAUSED`) —
pass `PAUSED` unless the user asked for it to go live.

Because configuration is spread across setters, **propose the whole
configuration as one plan** rather than narrating a dozen separate changes. The
user should approve a campaign, not twelve settings.

## Creative sets

Creative is grouped into sets, not attached per-ad:

`applovin_create_creative_set` → `applovin_upload_assets` →
`applovin_add_assets_to_creative_sets` → `applovin_attach_creative_set`

`applovin_clone_creative_set` to iterate on a winner,
`applovin_detach_creative_set` and `applovin_remove_assets_from_creative_sets`
to withdraw. `applovin_set_creative_localization` for per-market variants.

## Reporting — cohorts, not last click

This is where AppLovin differs most:

- `applovin_get_advertiser_report` — spend and installs
- **`applovin_get_roas_cohort_report`** — return by install cohort over time.
  Mobile UA is judged on what a cohort is worth at day 7 or 30, not on
  day-one ROAS. Use this before declaring a campaign good or bad.
- `applovin_get_retention_report` — whether installs stick
- `applovin_get_creative_performance` — which creative sets earn

A campaign with poor day-one ROAS and strong day-30 cohort return is working.
Judging it on the first number is the classic mobile UA mistake.

## Tracking

`applovin_list_tracking_integrations`, `applovin_set_tracking_configuration`.
Attribution runs through an MMP; without it the reports above are guesses.
