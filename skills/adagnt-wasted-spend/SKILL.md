---
name: adagnt-wasted-spend
description: Wasted-spend audit on the ad accounts a person connected — one ledger in which every unit of spend is wasted, too early to call, working or not judged, with the change that stops each leak sent for approval. Use for "where am I wasting money", cost-cutting passes, or monthly hygiene.
---

# AdAgnt Wasted-Spend Audit

The wasted-spend answer is AdAgnt's ledger. The tools compute it; report it as
it comes back. Do not build a second definition of waste in the chat.

## What the ledger says

- Every unit of spend in the window gets one verdict: **wasted**, **too early
  to call**, **working**, or **not judged**. The four add up to what the
  platform says was spent.
- **Wasted is proven, never assumed.** Something that took a share of the spend
  should have brought about that share of the account's own conversions. AdAgnt
  computes the chance of so few by luck; under 1 in 20, it is wasted.
- **Too early to call** is everything else with no conversions yet. The answer
  says how much more spend without a conversion would be proof.
- **An account that records no conversions is not judged.** The answer says so.
  The next step is conversion tracking (`audit_conversion_tracking`), never a
  pause.
- **Costs too much** (converts, at twice the account's cost per conversion or
  more) is not waste and is never proposed for a pause. It is a bid or budget
  question.
- Money is in the account's own currency, written with its code. Do not
  convert it and do not write a `$`.

Zero conversions, few clicks or a spend threshold are not proof. Never add
"waste" of your own to the ledger's numbers.

## Step 1 — Scope

Read `STRATEGY.md` for what the user chose to keep running on purpose, and
Performance History for earlier audits. Use the tool's default window unless
the user names one; a short window leaves more of the spend too early to call.

## Step 2 — Run the audit on each connected platform

- **Google Ads:** `analyze_wasted_spend`. With a `customer_id` it judges
  campaigns, search terms and the words inside them, read from Google's search
  terms report. Without one, when several accounts are connected, it reads
  every account (campaigns only) and returns one row per account.
- **Reddit Ads:** `reddit_analyze_wasted_spend`, campaigns, one account per call.
- **X Ads:** `x_ads_analyze_wasted_spend`, campaigns, one account per call.
- **Amazon Ads:** no wasted-spend tool, and the ledger does not judge Amazon
  yet. Amazon's reports are reports; call nothing in them wasted.
- **AppLovin:** no wasted-spend tool. Its campaign report carries the ledger's
  line; report that line.
- A platform that answers it is **not connected** or **coming soon** is not
  audited. Say so and move on.

An account the platform will not read comes back as one row with the
platform's reason in its own words. Report that row as written; the rest of the
answer stands.

## Step 3 — Report

Lead with the four numbers, as returned. Then:

1. Where it leaks: each leak with what it spent and the odds.
2. What is too early to call, with how much more spend would be proof.
3. What was not judged, and why.
4. What the tool says it did not read on this account.

## Step 4 — Stop a leak

A leak comes with the change that stops it, where an AdAgnt tool makes that
change: a pause for a campaign (`pause_campaign`), a negative keyword for a
search term (`add_negative_keywords`). It reaches the account owner as a
proposal; nothing changes until they approve it, and AdAgnt reads the platform
back afterwards. Where no tool makes the change, the answer says so; say the
same.

Blocking a term that converts costs revenue. When a term is ambiguous, ask.

## Step 5 — Log

Append a dated entry to STRATEGY.md → Performance History: the window, the four
numbers with the currency code, the changes the user approved, and a date to
look again.
