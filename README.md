# AdAgnt Ads Manager for Claude

AdAgnt is an ads manager you talk to. With this plugin, Claude can research
keywords with real search volumes and bid ranges, build complete campaigns,
report on performance across platforms in one answer, find the spend that
returns nothing, and move budget toward what converts.

Live accounts connect today for **Google Ads, Amazon Ads, Reddit Ads and X Ads**,
and **AppLovin** for reporting. Meta, TikTok and Snapchat run on demo data
while their API reviews finish. You start in a sandbox with realistic demo
accounts, so you can try everything with no ad account and no risk.

**You approve every live change.** On a live account, a change Claude asks for
is filed as a proposal with the current and proposed values. Nothing reaches
your ad account until you approve it in the AdAgnt app.

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
AdAgnt does not read your Claude conversations, memory or files. Details:
https://mcp.adagnt.com/privacy.

## Plans

Free: every ad account you run, unlimited reading, and 10 changes a month,
no card needed. Paid plans remove the change limit. See https://adagnt.com.

## Support

- Setup guide: https://adagnt.com/start/
- Support: support@adagnt.com · https://mcp.adagnt.com/support
- Terms: https://mcp.adagnt.com/terms · Privacy: https://mcp.adagnt.com/privacy
