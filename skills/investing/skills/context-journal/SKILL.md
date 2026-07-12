---
name: context-journal
description: Update the investor's living context after any session with decisions, trades, or new information. Use when the user says "I bought/sold/exited/invested", reports a P&L, changes a goal or date, adds a policy, or at the end of any session where portfolio state changed. Also use BEFORE trades to check against past lessons.
---

# Context Journal

The state-keeper. Two jobs: (A) keep ../../investor-context.md current; (B) grow ../../lessons.md and enforce it.

## A — Update ritual (after any change)
1. Read current investor-context.md.
2. Apply changes: holdings (recompute blended averages when tranches are added), goals, balances, insurance, newly articulated rules.
3. Update the "Last updated" date. Rewrite cleanly — the file must stay under ~1 page so any model ingests it fast.
4. Confirm to the user exactly what changed (diff-style summary).

## B — Lesson ritual (after any closed trade or notable miss)
Append to lessons.md: date, trade, outcome, and 1–3 extracted RULES as imperatives. Rules must be checkable (levels, timings, sizes), not vibes.

## C — Pre-trade gate (when any skill proposes a trade)
Scan lessons.md; if the proposal violates a recorded rule, quote the rule verbatim and require explicit user override before proceeding.

## Formatting rules
- investor-context.md: facts only. lessons.md: append-only, never rewrite history.
- Never store account numbers, passwords, or credentials.
