# AdAgnt Ads Manager for Claude

AdAgnt is an ads manager you talk to. With this plugin, Claude can research
keywords with real search volumes and bid ranges, build complete campaigns,
report on performance across platforms in one answer, find the spend that
returns nothing, and move budget toward what converts.

Accounts connect today for **Google Ads, Amazon Ads, Reddit Ads and X Ads**,
and **AppLovin** for reports. Meta, TikTok and Snapchat are coming soon.
AdAgnt answers from the ad accounts you connect. Until you connect one, it
says how; nothing is answered from sample data.

**Changes to an ad account wait for approval.** On a connected ad account, a
change Claude asks for is filed as a proposal with the current and proposed
values. Nothing reaches the ad account until it is approved, on the card in
the chat or in the AdAgnt app. AdAgnt then makes the change once, reads the
platform back and records it in a ledger.

What that does and does not cover:

- Claude cannot approve a proposal it filed itself.
- A proposal filed by AdAgnt's own automation (a monitor, or an autopilot
  sweep you asked for) waits in the same queue. You can approve it on the card
  or in the app, or tell Claude to approve it; Claude then calls
  `manage_action`, and that call makes the change on the ad platform.
- Tools that change only AdAgnt's own records (monitors, schedules, saved
  strategy and revenue assumptions, the to-do list, which account is primary)
  run when Claude calls them. They change nothing in an ad account.

## What's in the plugin

- **The AdAgnt connector** (`.mcp.json`): Claude connects to the AdAgnt MCP
  server at `https://mcp.adagnt.com/mcp`. The first time, Claude opens AdAgnt's
  sign-in page in your browser (email code or Google). Your ad platforms are
  connected separately at https://app.adagnt.com, each with its own sign-in.
- **16 skills** (`skills/`): how to run AdAgnt well. Account setup, the
  tool-call contract, campaign building, ad copy, performance reporting,
  wasted spend, strategy memory, revenue estimates, autopilot, and one skill
  per ad platform (Google, Meta, TikTok, Amazon, AppLovin, Snapchat).

## What it sends and where

The plugin runs no code on your machine. The skills are instructions Claude
reads. The only network destination is the AdAgnt server
(`https://mcp.adagnt.com`): Claude sends it the arguments of each AdAgnt tool
it calls, and AdAgnt calls the ad platforms you connected on your behalf.

- **Your files.** The AdAgnt server cannot read files on your machine. Some
  skills have Claude keep a `STRATEGY.md` in your workspace: Claude reads and
  writes that file there.
- **What is stored.** AdAgnt stores what its tools are given and what they
  change: your account email, the ad accounts you connect and their access
  tokens (encrypted), the business profile and strategy notes Claude saves
  with `save_business_profile` and `update_strategy` (what you sell, your
  audience, goals and budget guidance), and the ledger of every change with
  who asked and why.
- **Personal data in your ad accounts.** A tool that reads personal data held
  in an ad account (for example lead-form submissions on Meta, once Meta is
  live) returns it to Claude in the chat.
- AdAgnt does not read your Claude conversations or Claude's memory.

Details: https://mcp.adagnt.com/privacy

## Plans

Free: every ad account you run, 20 reads and 10 changes a month (no cap during launch),
no card needed. Paid plans read without limit and allow more changes; Agency
has no cap. Prices: https://adagnt.com/#pricing.

## Support

- Setup guide: https://adagnt.com/start/
- Support: support@adagnt.com · https://mcp.adagnt.com/support
- Terms: https://mcp.adagnt.com/terms · Privacy: https://mcp.adagnt.com/privacy
