---
name: branding-and-design-system
description: Create brand identity and a design-token system for a product. Use when the user asks for a name, logo direction, color palette, typography, "make it look good/professional", visual identity, style guide, or design system — or after product-and-ux-research completes and no design system exists.
---

# Branding & Design System

Import ../../agentic-engineering-discipline.md. Read prd.md first — brand must serve the personas, not the builder's taste.

## Procedure
1. **Brand core:** 3 adjectives the product should evoke (get user agreement). Naming: generate 8–10 candidates across styles (descriptive/abstract/compound); check domain + obvious trademark collisions via search; shortlist 3.
2. **Color system:** primary, secondary, accent, semantic (success/warn/error/info), neutrals scale (50–900). HARD RULE: verify WCAG AA contrast (4.5:1 body text, 3:1 large/UI) for every foreground/background pair you propose — state the ratios.
3. **Typography:** max 2 families (display + body). Modular scale (e.g., 1.25 ratio) → explicit px/rem sizes for h1–h6, body, small. Line-height and letter-spacing per size.
4. **Spacing & radii tokens:** 4px or 8px base grid; spacing scale; radius scale; shadow scale (3 levels max).
5. **Component inventory:** list only components the PRD flows actually need. No speculative component libraries (minimal-code principle).

## Output artifact — design tokens AS CODE
Emit tokens in the stack-appropriate format (CSS custom properties by default; adapt if the project uses another system). Frontend inherits mechanically — never re-invents values inline.
Also save brand.md: adjectives, name rationale, palette with hex + contrast ratios, type scale table, usage do/don'ts.

## Rules
- Dark mode: decide day one (tokens make it cheap; retrofitting is expensive). Ask the user.
- Accessibility is not optional polish; failing contrast = rejected palette.
