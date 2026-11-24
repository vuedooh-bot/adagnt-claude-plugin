---
name: adagnt-mcp
description: The AdAgnt tool-call contract — how to resolve accounts, the hard argument limits that reject calls, what each error code means, and how quota works. Load this before calling AdAgnt tools so calls succeed first time instead of failing validation.
---

# AdAgnt tool-call contract

352 tools across eight ad platforms. Most failed calls are not hard problems —
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

AdAgnt answers only from the ad accounts the person connected. Nothing is
answered from sample data. When a platform cannot answer you get `NO_ACCOUNT`,
and the message says which of four things it is:

- **Not connected** — the message carries the Connect link (it opens that
  platform's own sign-in). Give the user the link, or call
  `get_connections_status` to show the card.
- **The login has no ad accounts** — the message says which login to use.
- **The platform did not answer just now** — ask again in a few minutes.
- **Coming soon** — "Meta Ads is coming soon to AdAgnt. Today AdAgnt works
  with …". That is the whole answer for that platform: say it, name the
  platforms that work today, and stop.

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

- **`get_roas_estimates` throws without saved revenue assumptions.** Check
  `list_revenue_sources` first; if empty, propose `set_revenue_assumptions`
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
| `NO_ACCOUNT` | The platform cannot answer for this person: not connected, a login with no ad accounts, the platform did not answer, or the platform is coming soon. The message says which. | Report the message as written (it carries the Connect link when there is one). Do not retry, and do not answer from memory or examples instead. |
| `NOT_SUPPORTED` | The operation is not available on this driver | Not a bug. Say so plainly and stop. |
| `PLATFORM_ERROR` | The ad platform itself rejected the call | Report the platform's reason. Do not retry a write. |
| `QUOTA_EXCEEDED` | A monthly allowance was reached (free plan: 20 reads and 10 changes a month; no cap during launch, so not returned until launch ends) | Report it. `get_usage_status` shows what is left and the reset date. |

**Known `NOT_SUPPORTED`: AppLovin on a live account.** All writes throw, plus
creative-set listing, asset listing and targeting search. Campaign reads and
the whole reporting surface work. Tell the user which they are hitting rather
than implying a bug.

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

Every plan, free included, connects any number of ad accounts. The free plan
allows 20 reads (reports, analysis, diagnostics) and 10 writes a month; paid
plans read without limit and allow more writes (Agency has no cap). During
launch no plan is capped: calls are counted and nothing is refused, and
`get_usage_status` says so (`limits_enforced: false`). `get_usage_status` returns the plan, this month's reads and
writes so far, how many are left (null = no limit on this plan), the reset
date, and the plans above. When an allowance is hit, the tool stops cleanly —
nothing is left half-changed.

## 7. Results

Every tool returns a JSON object, and the MCP layer sends it as
`structuredContent` alongside the text. Read fields directly rather than
re-parsing the text block.
