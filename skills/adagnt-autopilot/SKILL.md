---
name: adagnt-autopilot
description: Run AdAgnt's one-command optimization sweep across every connected ad platform, review the proposed actions it files to the approval queue, and approve or reject each one with the user. Use when the user wants a full account check-up, says "run autopilot", or asks what AdAgnt recommends changing.
---

# AdAgnt Autopilot

One command sweeps all connected platforms for problems and files proposed fixes to an approval queue. The whole point of the design: **autopilot proposes, the user disposes.** Nothing changes an ad account until a human approves it.

## Non-negotiable safety rules

- **Never approve a pending action without the user's explicit consent.** Not "the user seemed to want this", not "it's obviously right", not batch-approving because they approved the last three. One explicit "yes" per action, or an explicit "approve all" after they have seen the full list.
- **`manage_action` with an approve decision executes a real write** — budget changes, pauses, creative changes. Treat it with the same one-call-then-verify discipline as any write tool.
- If the user is not present to review, stop after presenting the queue. The queue keeps; it does not need to be drained in one sitting.

## When to run `run_autopilot_cycle`

Good triggers:

- The user asks for a check-up: "how's everything looking", "anything I should fix", "run the sweep"
- A recurring cadence the user has asked for (weekly is typical; daily only on high-spend accounts)
- After a burst of changes — new campaigns, budget moves — to catch anything the changes broke
- When a monitor fired and the user wants the full picture, not just that one alert

Pass `focus` when the user's question is narrow: `wasted_spend`, `budgets`, or `creative`. Omit it (or use `all`) for the general sweep. The call evaluates all monitors, hunts wasted spend, checks budget efficiency and creative fatigue, files prioritized proposed actions to the queue, and returns an executive brief.

Present the brief as-is first — headline findings, biggest dollar figures, what it proposed — before diving into the queue.

## Reviewing the queue — `list_pending_actions`

Call `list_pending_actions` and walk the user through what's waiting. For each action, present:

1. **What it will do** — the exact change (pause campaign X, move $Y from A to B)
2. **Why it was proposed** — the evidence in the action's rationale
3. **What it costs to be wrong** — a paused campaign can be resumed; a spent budget cannot be un-spent

Order the walkthrough by dollar impact, largest first. If an action's rationale looks stale (the campaign was already fixed, the data window predates a change the user made), say so and recommend rejecting it.

## Deciding — `manage_action`

For each action, get one of three answers from the user:

- **Approve** — `manage_action` with the action id and an approve decision. Call it once. Then verify the change landed with the matching read tool (`get_campaign_performance`, `list_campaigns`, or the platform equivalent) and echo what actually changed.
- **Reject** — `manage_action` with a reject decision. Note why, so the same proposal isn't relitigated next cycle.
- **Defer** — leave it in the queue. Fine. Say when it will come up again.

If the user says "approve everything": read the full list back to them first, with totals ("6 actions, net $140/day of budget moves, 2 pauses"), and get one confirmation of that summary. Then approve one at a time, verifying each.

## Standing watch — monitors

Autopilot cycles are point-in-time; monitors watch continuously between them:

- `create_monitor` for the metrics the user actually cares about (CPA ceiling, spend spike, CTR floor). Tie thresholds to targets in strategy memory, not round numbers.
- `test_monitor` immediately after creating one — a monitor that never fires and a monitor that fires hourly are both useless, and testing is how you find out which you built.
- Monitor-initiated proposals land in the same `pending_actions` queue. Same rule applies: no approval without the user.

## Wrap up

After a cycle, record the outcome to strategy memory with `update_strategy` (`append_learning` is enough: date, what was approved/rejected, expected effect). Next cycle should read it and not re-propose what the user already rejected.
