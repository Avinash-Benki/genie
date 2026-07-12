---
name: spec-and-context-engineering
description: Write specs that AI agents can execute reliably, author project CLAUDE.md files, and choose the right agent workflow (plan-execute-review vs autonomous loops). Use when the user wants to delegate a coding task to an agent, asks "write a spec/prompt for this feature", sets up a new repo for AI-assisted work, or an agent session has gone off the rails.
---

# Spec & Context Engineering

Import ../../agentic-engineering-discipline.md. This is the meta-skill: the quality of agent output is bounded by the quality of the spec and context you give it.

## Executable spec template (per task/feature)
1. **Goal** — one sentence, testable.
2. **Context** — files/modules involved; what already exists; what must NOT change (the surgical boundary).
3. **Constraints** — stack, conventions, performance/security requirements.
4. **Acceptance criteria** — numbered, individually verifiable; these become the tests.
5. **Out of scope** — explicit list; the primary defense against agent scope creep.
6. **Verification** — the exact command(s) that must pass.
A spec missing acceptance criteria or out-of-scope is not ready to hand to an agent.

## Project CLAUDE.md authoring
Contents (keep under ~1 page): stack + versions · directory map (3 levels) · commands (dev, test, lint, build) · conventions (naming, error handling, state patterns) · things agents must never do in this repo (e.g., touch migrations, edit generated files) · links to prd.md / brand.md / ops.md.
Update it whenever a convention changes — stale CLAUDE.md is worse than none.

## Workflow selection
| Situation | Workflow |
|---|---|
| Ambiguous or novel problem | Plan-execute-review: agent proposes plan, human approves, then implements slice by slice. |
| Well-specified, test-covered task | Supervised execution: agent runs, human reviews diff + green tests. |
| Bulk mechanical change (rename, codemod) | Autonomous loop on a branch; tests as fitness function; human reviews the final diff only. |
| Throwaway prototype/demo | Vibe mode is fine — declare it, and never promote vibe output to production without a real review pass. |

## Context hygiene
- One task per session where possible; summarize decisions into the spec/CLAUDE.md and reset long sessions instead of dragging rotten context.
- Paste errors verbatim; never paraphrase stack traces to an agent.
- When an agent loops on a bug >3 attempts: stop, extract a minimal repro, restate the problem fresh (often in a new session).
