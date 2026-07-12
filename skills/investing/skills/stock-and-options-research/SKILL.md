---
name: stock-and-options-research
description: Multi-framework Indian stock research and options trade construction. Use when the user asks for stock picks, stock analysis, "which stocks to buy", theme/sector analysis (EV, AI, power, defence), value-chain mapping, PEAD trades, or any options strategy (spreads, strangles, iron condors) on Nifty/Bank Nifty/stocks.
---

# Stock & Options Research

ALWAYS read ../../investor-context.md AND ../../lessons.md first. Run market-pulse if fresh money is involved.

## Part A — Six-framework stock analysis
Run ALL applicable lenses; a stock scoring in 2+ frameworks ranks higher.
1. **Deep value:** P/E < 15 with ROE > 20%, low debt, positive FCF, dividend support. Distinguish genuine value from value traps (cyclical earnings peak = trap; BPCL-at-high-crude lesson).
2. **Momentum/PEAD:** relative strength vs Nifty (4-week), earnings upgrades, post-results drift. PEAD entry ONLY after results confirm beat quality (revenue + PAT + guidance, not just PAT). Size ₹6–10K, 7% stop.
3. **GARP:** PEG < 1, 15%+ revenue CAGR at historical-discount valuations.
4. **Theme investing:** map the ENTIRE value chain (raw material → components → OEM → services). Core insight: the components/picks-and-shovels layer usually beats OEMs — confirmed volumes, diversified customers, better pricing power.
5. **First principles:** decompose the sector's economics; find which layer captures margin (e.g., CDMO 25–45% EBITDA in the chemicals chain).
6. **Contrarian:** maximum-pessimism sectors where data contradicts narrative. Require a concrete catalyst + a valuation floor (dividend yield or book value).

### Output format (mandatory)
Ranked table: Stock | CMP (searched, dated) | Framework(s) | 6M target (analyst-sourced, cited) | Stop loss | Catalyst + date | Conviction (VH/H/M).
Then: allocation respecting the user's sizing rules; risks; what would invalidate each thesis.

## Part B — Options playbook (hard rules from real losses)
- Defined-risk only (spreads, iron condors). No naked shorts.
- Gap-up days: sell put spreads 300–400 pts BELOW spot, never ATM (Apr-2026 pinning loss).
- Never enter in the first 15–20 minutes of the session.
- Gates before recommending: R:R ≥ 0.5; POP ≥ 60% for credit strategies; margin ≤ available funds (ask user); no RBI/major-results/geo event inside the trade's life unless it is explicitly an event trade.
- Exit discipline: at 7–10% of max loss, exit. At 50–60% of max profit on credit spreads, book.
- Strangle activation template (ITC pattern): require N stable sessions above a defined level + elevated IV; a single event-driven pop does NOT qualify.
- Before ANY trade: check lessons.md — does this violate a recorded lesson? If yes, say so and stop.

## Honesty rules
- Search live prices; never quote stale CMPs as current.
- Present the bear case with equal effort. If the user already holds correlated exposure (check context file), flag concentration instead of adding.
