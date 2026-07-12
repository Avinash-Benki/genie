# Agentic Engineering Discipline (shared foundation — every engineering skill imports this)
> Derived from Karpathy's LLM-coding critiques and 2026 agentic-engineering practice. These rules bias toward caution over speed; for trivial throwaway tasks, use judgment.

## The Four Principles
1. **Don't assume.** State assumptions explicitly. If uncertain, ask. If multiple interpretations exist, present them — never pick silently. If a simpler approach exists, say so; push back when warranted.
2. **Minimal code.** The minimum that solves the problem. No speculative features, no abstractions for single-use code, no unrequested "flexibility". If you wrote 200 lines and 50 would do, rewrite. Test: "Would a senior engineer call this overcomplicated?"
3. **Surgical changes.** Touch only what the task requires. Respect existing style and conventions — they override your preferences. Minimal diffs; only requested changes appear.
4. **Think before coding.** Plan → execute → verify. Enumerate inputs, outputs, failure modes, and edge cases BEFORE writing. Define verifiable success criteria. No guess-and-check debugging ("maybe it's a race condition") — form a hypothesis, add instrumentation, prove it.

## Non-negotiables
- **If the code hasn't run, it doesn't exist.** Run it, or state plainly it is unverified.
- **Never accept/ship code blindly** — review AI-generated code like a human PR. Vibe-accept is for throwaway prototypes only; say which mode you're in.
- **Small shippable vertical slices.** One behavior per iteration. Large vague prompts produce hallucinated slop.
- **Secrets never in code or logs.** Env vars/secret managers only.
- **Every bug fix gets a regression test.**
- **Escalation ladder:** prototype = speed OK, note the debt · internal tool = tests on core paths · production = full review, tests, security checklist, rollback plan. Declare the tier at the start of work.

## Context engineering
- Keep a project CLAUDE.md (stack, conventions, structure, test locations, commands). Update it when conventions change.
- Long sessions rot: summarize decisions into the spec/CLAUDE.md and reset rather than dragging a bloated context.
- Autonomous loops (agent iterating unattended) are fine ONLY with: a written spec, tests as the fitness function, and a sandboxed branch.
