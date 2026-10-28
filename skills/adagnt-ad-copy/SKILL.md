---
name: adagnt-ad-copy
description: Write and update ad creative that fits platform limits and the workspace brand voice — Google RSA headlines/descriptions, Meta primary text, TikTok ad text. Use when drafting new copy or refreshing underperforming ads.
---

# AdAgnt Ad Copy

Produce copy that (a) fits the platform's hard limits, (b) sounds like the brand, and (c) gives the ad-serving algorithm enough distinct material to optimize with.

## Step 1 — Ground in brand voice

Read `STRATEGY.md` before writing a single line. Use:

- **Brand Voice** — tone, words to prefer, words to ban, claims that are/aren't allowed
- **Business Profile** — the actual value proposition and proof points
- **Target Audience** — pains and motivations to lead with

If the section is thin, ask the user three quick questions (tone in three adjectives, one claim they're proudest of, one phrase they never want in an ad) rather than guessing. `suggest_ad_content` can draft candidates from `key_benefits`, `pain_points`, and `proof_points` — treat its output as raw material to edit into brand voice, not final copy.

## Hard limits (count characters before presenting anything)

| Platform | Field | Limit |
|---|---|---|
| Google RSA | Headline | **30 chars**, up to 15 headlines (min 3) |
| Google RSA | Description | **90 chars**, up to 4 descriptions (min 2) |
| Google | Callout | 25 chars, no CTAs, no trailing period |
| Google | Sitelink text | 25 chars |
| Meta | Primary text | ~125 chars visible before truncation |
| Meta | Headline | 40 chars |
| Meta | Description | 30 chars |
| TikTok | Ad text | 100 chars |

If any line is over limit, fix it before showing the user. Show character counts next to each line.

## RSA construction rules (Google)

- Supply 8–15 headlines that are genuinely different from each other — vary the angle: value prop, pain point, proof/social trust, offer, CTA, brand. Near-duplicates waste slots.
- At least a few headlines should carry the primary keyword of the ad group; the rest earn attention.
- Avoid pinning unless a compliance line must always show — pinning constrains the combinatorial optimizer.
- Descriptions: 2–4, each a complete thought that works after any headline. Don't end headlines with punctuation; don't SHOUT.
- No superlatives you can't substantiate, no "#1" without a source, and respect any regulated-vertical rules noted in STRATEGY.md.

## Meta / TikTok notes

- **Meta**: front-load the hook in the first sentence of primary text; the headline carries the offer. Provide 2–3 primary-text variants for testing. For DCO use `create_meta_dco_ad` with multiple assets per slot.
- **TikTok**: conversational and native — copy supports the video, it doesn't carry the ad. Avoid corporate phrasing entirely.

## Step 2 — Present before writing

Show the full copy set in a table with character counts and a one-line rationale per angle. Get explicit approval — creative updates modify live ads and reset learning in some cases.

## Step 3 — Apply (one call, verify, no auto-retry)

- Google: `update_ad_headlines` / `update_ad_descriptions` (pass `customer_id`, `ad_group_id`, `ad_id`, and the full replacement list — these replace, not append). Broader edits: `update_ad_content`. New ads: `create_ad`. Extensions: `add_callout_extensions`, `add_sitelinks`, `add_structured_snippets`.
- Meta: `update_meta_ad`; inspect current creative first with `get_meta_ad_creatives`.
- TikTok: `update_tiktok_ad_group` for placement-level changes; new creative via `add_tiktok_ad`.

Each of these is a paid-account write: call once, and if it errors, verify actual state with the matching read tool before considering another attempt.

## Step 4 — Log it

Append to STRATEGY.md → Performance History: date, ad IDs touched, what changed, and the hypothesis (e.g. "pain-point headlines to lift CTR on ad group X"). When a copy test concludes, record the winner and why under Brand Voice so future copy starts smarter.
