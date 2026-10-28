---
name: adagnt-mcp
description: The AdAgnt tool-call contract — how to resolve accounts, the hard argument limits that reject calls, what each error code means, and how quota works. Load this before calling AdAgnt tools so calls succeed first time instead of failing validation.
---

# AdAgnt tool-call contract

348 tools across eight ad platforms. Most failed calls are not hard problems —
they are the same handful of avoidable mistakes. This is that list.

## 1. Resolve the account before anything else

Tools operate on the user's *primary* account for a platform unless told
otherwise.

- `list_connected_accounts` — what is connected, and which is primary
- `get_connections_status` — whether a platform is linked and healthy
- `switch_primary_account` — change the default (a write)

**Amazon is different.** Every Amazon tool needs a **profile**, which pairs an
advertiser account with one marketplace. Call `amazon_list_profiles` first and
pass the profile explicitly. A US profile cannot see UK campaigns.

**AppLovin** uses accounts from `applovin_list_accounts`.

If no account exists you get `NO_ACCOUNT`. In sandbox that usually means the
platform has not been touched yet — a demo account with 90 days of history is
created the first time you use a platform, so simply proceeding will seed it.

## 2. Hard argument limits that reject the call

These are enforced by schema. Getting them wrong returns `INVALID_ARGS` and
nothing is created.

| Tool | Field | Limit |
|---|---|---|
| `create_search_campaign` | `ad_groups[].headlines` | **exactly 15** |
| `create_search_campaign` | `ad_groups[].descriptions` | **exactly 4** |
| `create_search_campaign` | `ad_groups[].keywords` | **at least 5** |
| `create_ad` | `headlines` | 3–15 |
| `update_ad_headlines` | `headlines` | 3–15 |

**Write the full set yourself.** Do not ask the user for fifteen headlines, and
never promise them a smaller number — Google itself allows 3–15 headlines and
2–4 descriptions, so this constraint is AdAgnt's and stricter than the platform's.

## 3. Order of operations that actually matters

- **`get_true_roas` throws without saved revenue assumptions.** Check
  `list_revenue_sources` first; if empty, propose `connect_revenue_source`
  (a provider label plus the user's average order value — nothing is connected
  to GA4, Shopify, Stripe or Klaviyo; revenue is modelled as conversions ×
  order value over 90 days) and wait for approval. Do not call it and report
  the error as a finding.
- **Keyword research before campaign creation.** `research_keywords` returns
  real volumes, competition and bid ranges. Inventing keywords wastes the
  user's money.
- **Assets before campaigns** on Meta and TikTok —
  `validate_and_prepare_*_assets`, `upload_tiktok_images`, `validate_video`.
- **Amazon:** profile → campaign → ad group → product ad → keywords. Each step
  needs the ID from the one before.

## 4. Error codes, and what each one actually means

| Code | Meaning | What to do |
|---|---|---|
| `INVALID_ARGS` | Arguments failed validation | Read the message — it names the field. Fix and retry **once**. |
| `NOT_FOUND` | The named object does not exist | List first, then act on a real ID. Never guess an ID. |
| `NO_ACCOUNT` | No connected account for that platform | Offer to connect it. |
| `NOT_SUPPORTED` | The operation is not available on this driver | Not a bug. Say so plainly and stop. |
| `PLATFORM_ERROR` | The ad platform itself rejected the call | Report the platform's reason. Do not retry a write. |
| `QUOTA_EXCEEDED` | Monthly write allowance reached (free plan: 10 changes a month) | Report it; reads still work. `get_usage_status` shows plan and reset date. |

**Known `NOT_SUPPORTED`: AppLovin on a live account.** All writes throw, plus
creative-set listing, asset listing and targeting search. Campaign reads and
the whole reporting surface work. In sandbox all 36 AppLovin tools work. Tell
the user which they are hitting rather than implying a bug.

## 5. Writes

- Every write tool is annotated **destructive**, so ChatGPT raises its own
  permission prompt. That prompt describes intent, not values — it is not a
  substitute for showing the user the actual budget, keywords and ad copy.
- **Never auto-retry a write after an ambiguous failure.** That is how
  duplicate campaigns and double spend happen. Retry only `INVALID_ARGS`, which
  provably did nothing.
- **Verify with an independent read.** Do not report success or failure from
  the write's own return value alone. A write here has come back looking like a
  failure while the campaign was in fact created — reporting that verbatim
  would have had the user create it a second time. Call the matching list/get
  tool and report what you observed there.

## 6. Quota

Only writes are metered. Every plan, free included, connects any number of ad
accounts, and reads (reports, analysis, diagnostics) are never counted. The
free plan allows 10 writes a month; paid is unlimited. `get_usage_status`
returns the plan, the monthly write limit (null = unlimited), the ad-account
limit (null = unlimited on every plan), the reset date, and the upgrade
options. When the write allowance is hit, write tools stop cleanly — nothing is
left half-changed, and reads keep working.

## 7. Results

Every tool returns a JSON object, and the MCP layer sends it as
`structuredContent` alongside the text. Read fields directly rather than
re-parsing the text block.
