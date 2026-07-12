---
name: frontend-engineering
description: Build frontend/UI code with quality guardrails. Use when the user asks to build a UI, page, component, web app frontend, fix layout/styling, improve performance or accessibility, or wire a frontend to an API. Framework-neutral.
---

# Frontend Engineering

Import ../../agentic-engineering-discipline.md. Read prd.md + brand.md/tokens if they exist; if not, flag the gap and either run upstream skills or record assumptions.

## Framework selection (when green-field; otherwise respect the existing stack absolutely)
Decide by: team familiarity > ecosystem need (SSR? SEO? realtime?) > simplicity. State the decision and 2-line rationale. A static page needs no framework; don't scaffold an SPA for a brochure.

## Architecture rules
- Components: small, single-purpose; container/presentation separation only when state complexity demands it.
- State decision tree: local state → lift to parent → context/shared store ONLY when prop-drilling exceeds ~3 levels or state is truly global (auth, theme). No global store by default.
- Data fetching: centralize API calls in one client module; handle loading/error/empty states for EVERY fetch — a component without those three states is incomplete.
- Use design tokens for all values; inline magic numbers/colors are defects.

## Quality gates (check before declaring done)
- **Semantic HTML:** headings hierarchy, button vs div-onclick, form labels, landmarks. Divs-for-everything = reject.
- **Accessibility:** keyboard navigable, focus visible, alt text, aria only where semantics can't do it.
- **Responsive:** mobile-first; test the layout at 360px, 768px, 1280px conceptually and state how each breaks.
- **Performance budget:** name it (default: LCP < 2.5s, CLS < 0.1, bundle < 200KB gz for MVPs); lazy-load below-the-fold; images sized + modern formats.
- **If the code hasn't rendered, it doesn't exist** — run it or say it's unverified.

## Anti-slop rules
- No component libraries pulled in for one button. No CSS frameworks added to fix one margin. Respect the minimal-code principle.
- Never restyle or refactor code outside the requested change (surgical principle).
