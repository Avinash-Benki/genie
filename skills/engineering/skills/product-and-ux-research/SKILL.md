---
name: product-and-ux-research
description: Product discovery and UX research before building anything. Use when the user has an app/product idea, asks to "build X", mentions target users, competitors, features, user flows, wireframes, MVP scoping, or when no PRD exists yet for what they want built.
---

# Product & UX Research

Import ../../agentic-engineering-discipline.md. GATE: no downstream skill (frontend/backend/DB) should start building without this skill's output artifact existing.

## Procedure
1. **Problem statement:** one sentence — who has what pain, and what changes if solved. If the user can't fill this, stop and work it out with them.
2. **Users & JTBD:** 1–3 personas max (name, context, job-to-be-done, current workaround). Reject persona bloat.
3. **Competitor teardown:** search 3–5 existing solutions; table of what they do well / gaps / pricing. The gap column drives differentiation.
4. **Scope the MVP ruthlessly:** MoSCoW list. The Must column must fit in one buildable vertical slice. Everything else explicitly OUT of v1 (write the out-of-scope list — it prevents agent scope creep later).
5. **User flows:** text-based flow for each Must feature (screen → action → result → error path). Error paths are mandatory, not optional.
6. **Wireframe-in-text:** per screen — layout regions, components, primary action, empty/loading/error states.
7. **Success metrics:** 2–3 measurable (activation, task completion, retention proxy).

## Output artifact — PRD-lite (save as prd.md in the project)
Problem · Personas · Competitor gaps · MVP scope (in/out) · Flows · Screen specs · Metrics · Open questions.
Keep under 2 pages. Downstream skills read this file, not the chat history.

## Rules
- Ask before assuming platform (web/mobile/both), audience size, or monetization.
- Challenge feature-first thinking: every feature must map to a persona's JTBD or it goes to out-of-scope.
