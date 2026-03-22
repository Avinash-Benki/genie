# Design System Prompts

## Create a design System
```
This is a dashboard of a modern voice ai intelligence app called <Project>. 
Thoroughly analyze the UI in this screenshot and describe it in as much detail as you can to hand over from a UI designer to a developer. The brief should cover both light and dark mode and contain responsive breakpoints matching Tailwind CSS defaults. Output characteristics as structured JSONC. For colors, extract a rough palette and only detail accents and complex media. The goal is to use only 2 palettes: primary and secondary similar to Tailwind colors. Alongside these 2, you can define any number of grays and accent colors for more complex UI (gradients, shadows, SVGs, etc.). End with a prompt explaining how to implement the UI for a developer, but don't mention any tech specs; only a brief of the UI to be implemented and the token rules + usage. Output the prompt as a Markdown code block.
```

## Reverse Engineer 

```
### Role: Senior Design Systems Engineer & UI Archaeologist
### Task: Comprehensive UI Deconstruction & Developer Handoff

I am providing a screenshot of the "<Project>" <dashboard details>. Your goal is to reverse-engineer the entire design system and provide a technical implementation brief that allows a developer to recreate this UI perfectly using Tailwind CSS logic.

---

### PHASE 1: ATOMIC TOKEN EXTRACTION (Output as JSONC)
Analyze the UI and extract the design tokens. 
1. **Color Palette:** - Define a 'Primary' and 'Secondary' ramp (50-900) based on the image.
   - Define a 'Neutral' slate/gray scale for backgrounds and text.
   - Detail specific 'Semantic' colors (Success, Error, Warning, AI-Active).
   - Detail 'Complex Media': Extract hex codes and stop-points for all gradients, glassmorphism (blur/opacity), and glow effects (common in Voice AI).
2. **Typography:** Identify font pairings (Heading vs. Body), estimated weights, and a modular scale (base 16px).
3. **Effects & Elevation:** Describe the shadow system (X, Y, Blur, Spread) and border-radius constants.

---

### PHASE 2: COMPONENT ARCHITECTURE
Break down the UI into logical sections. For each, describe:
- **Layout Logic:** Is it a Grid or Flexbox? What is the `gap`?
- **Voice Specifics:** Analyze how audio waveforms, microphone status, and transcription bubbles are styled.
- **Micro-Interactions:** Based on the visuals, describe how buttons or cards should behave on :hover or :active states.

---

### PHASE 3: RESPONSIVENESS
Define how this dashboard transitions across Tailwind CSS default breakpoints:
- **Mobile (sm):** What stacks? What is hidden?
- **Tablet (md):** Does the sidebar collapse?
- **Desktop (lg/xl):** Maximum container widths and multi-column behavior.

---

### PHASE 4: THE DEVELOPER BRIEF (Markdown Code Block)
End your response with a final Markdown code block. This block should be a prompt intended for a Frontend Developer. 
- **The Brief:** Explain the "Visual Language" (e.g., "Minimalist Dark Mode with Neon Accents").
- **Token Usage:** Provide the rules for using the Primary/Secondary palettes.
- **The Challenge:** Tell the developer to implement the UI layout using the tokens provided above, emphasizing "Clean Code," "Accessibility," and "Fluid Transitions." 
- **Constraint:** Do not mention specific libraries (React/Vue/etc.); focus only on the UI characteristics and token rules.
```