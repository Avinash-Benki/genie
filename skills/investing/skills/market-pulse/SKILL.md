---
name: market-pulse
description: Run the pre-decision market snapshot ritual before any Indian investment decision. Use when the user asks "how's the market", "should I deploy now", "check market conditions", mentions Nifty/crude/VIX/FII levels, or before ANY skill recommends deploying fresh money into equity.
---

# Market Pulse

Purpose: produce a standardized snapshot + go/no-go verdict using the investor's own thresholds. ALWAYS read ../../investor-context.md first.

## Procedure
1. Web-search CURRENT values (never rely on training data): Nifty 50 & Sensex level and day move; Brent crude; India VIX; FII/DII net flows (latest session + MTD); USD/INR; dominant macro/geopolitical driver; upcoming events next 14 days (RBI MPC, expiry, major results, global events).
2. Render the snapshot table (all rows, with date/time of data).
3. Apply threshold rules IN ORDER; first match wins the headline verdict.

## Threshold rules (user-specific)
| Condition | Verdict |
|---|---|
| VIX > 20 | NO lump sums. SIPs continue. Option selling attractive but half size. |
| Crude > $100 | Avoid OMCs, paints, aviation, tyres. Favor upstream (ONGC), IT, pharma. |
| Crude < $85 | OMC re-rating window; check HPCL/BPCL momentum before entry (no chasing >8% day moves). |
| FII net sellers >10 sessions AND DII absorbing | Range-bound regime: deploy only at lower band of 30-day range. |
| FII flip to net buyers after long selling streak | Early re-rating signal: large-cap private banks first beneficiary. |
| Nifty < user's stated dry-powder trigger (see context file) | Deploy reserved tranche per plan. |
| Event <7 days away (RBI/major results) | No new options positions; IV crush risk. |
| None triggered | Normal regime: follow existing tranche plan. |

## Output format
Snapshot table → 2–3 sentence regime read → verdict line: **GO / GO-WITH-CONDITIONS / HOLD** → if conditions, name them explicitly (level + action).

## Hard rules
- Cite sources for every number; state data timestamp.
- Never say "markets will"; say "historically this regime has".
- If data conflicts across sources, run more searches; don't average silently.
