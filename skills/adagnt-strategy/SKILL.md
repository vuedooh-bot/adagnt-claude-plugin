---
name: adagnt-strategy
description: Create and maintain STRATEGY.md, the workspace's persistent marketing memory — six canonical sections read before every campaign action and updated with learnings after. Use when initializing strategy, updating it after results, or when any other skill finds it missing.
---

# AdAgnt Strategy Document

`STRATEGY.md` (workspace root) is the single source of truth that makes every session strategy-aware instead of starting from zero. Two rules govern it:

1. **Read before acting.** Any skill that researches, creates, edits, or reviews campaigns loads STRATEGY.md first and honors it.
2. **Write back after learning.** Campaign launches, performance reviews, audits, and concluded tests each append what was learned.

## Canonical structure — exactly these six sections

```markdown
# Marketing Strategy — {Business Name}
_Last updated: {YYYY-MM-DD}_

## 1. Business Profile
What is sold, to whom, at what price point; the core value proposition;
website; vertical; business model (e-com, SaaS, lead-gen, local…);
seasonality; regulated-vertical constraints if any.

## 2. Target Audience
Primary and secondary segments: who they are, the pain that brings them,
where they search/scroll, objections, and the language they use.
Geographic scope. Explicit exclusions (who NOT to target).

## 3. Competitive Landscape
Main competitors and how each positions; our differentiation;
whether we bid on competitor brand terms (and whether they bid on ours);
benchmark context worth remembering.

## 4. Campaign Strategy
Goals as numbers: target CPA and/or ROAS, monthly budget and its split
across platforms; which platforms and campaign types we run and why;
naming convention; standing rules (e.g. "never exceed $X/day without
asking", "always exclude existing customers").

## 5. Brand Voice
Tone in a few adjectives; words and claims to prefer; words and claims
that are banned; compliance requirements; approved proof points
(reviews, awards, stats with sources); winning copy patterns from tests.

## 6. Performance History
Append-only dated log, newest first. One entry per event:
- **{YYYY-MM-DD} — {launch|review|audit|test|change}:** what happened,
  IDs involved, the numbers, the lesson, next check-in date.
```

## Creating STRATEGY.md

1. Gather from the server first: `infer_business_profile` (derives a profile from Google account history), `get_business_profile` (anything previously saved), `get_benchmark_context` (vertical benchmarks for section 3), `list_campaigns` + platform equivalents (what already runs, for section 4).
2. Interview the user for what tools cannot know: pricing, differentiation, brand voice, budget, targets. Ask focused questions; do not fabricate placeholders that look like facts.
3. Write the file, marking anything unconfirmed as `TBD — confirm with user`.
4. Sync durable facts server-side with `save_business_profile` (`business_vertical`, `business_size`, `target_audience`, `geographic_focus`, `primary_goal`, `seasonality`) so server-side tools see the same profile.

## Maintaining it

- **After a campaign launch** (`adagnt-campaigns`): log the launch with IDs, budget, hypothesis, review date.
- **After a performance review** (`adagnt-performance`): log numbers vs targets, anomalies explained, actions taken. If a target changed, update section 4 — history logs events, but current truth lives in sections 1–5.
- **After a waste audit** (`adagnt-wasted-spend`): log waste found, cuts made, expected savings.
- **After a copy test concludes** (`adagnt-ad-copy`): move the winning pattern into section 5.
- Keep Performance History readable: one entry per event, newest first; when it grows past ~30 entries, compress the oldest into a short "lessons so far" digest at the bottom of the section.
- Update the `Last updated` date on every edit. Never silently rewrite history entries — append corrections instead.

## Conflict rule

If a user request contradicts STRATEGY.md (budget cap, banned claim, excluded audience), pause and surface the conflict: quote the strategy line, ask whether to override once or to update the document. The strategy is the user's own prior decision — protect them from accidentally violating it, but the user always wins.
