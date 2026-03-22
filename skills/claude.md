# Claude skills

Reference repos, docs, and learning material for [Claude Skills](https://docs.anthropic.com/en/docs/claude-code/skills) (portable `SKILL.md` packages used with Claude Code and compatible clients).

## Skill repositories

| Resource | Description |
|----------|-------------|
| [avoid-ai-writing](https://github.com/conorbronsdon/avoid-ai-writing) | Claude Code skill that audits and rewrites text to strip “AI-isms”—templated openers, filler, and inflated phrasing—with structured audits and replacement tables. |
| [superpowers](https://github.com/obra/superpowers) | Curated skill pack for agentic dev workflows: planning, TDD, debugging, code review, and similar routines as composable `SKILL.md` modules. |
| [context7](https://github.com/upstash/context7) | MCP server that pulls up-to-date library docs into the model context—pairs well with skills that need correct APIs and versions while coding. |

### More community & examples

| Resource | Description |
|----------|-------------|
| [GitHub topic: claude-skills](https://github.com/topics/claude-skills) | Topic listing—filter by stars/recency to discover new skills (quality varies; review before use). |
| [ComposioHQ awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | Curated list of skills and related tooling (verify repos before trusting in production). |

## Building skills

| Resource | Description |
|----------|-------------|
| [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) | Anthropic PDF: how Skills work, structure of `SKILL.md`, discovery, and best practices for writing effective, reusable skills. |

### Official docs & patterns

| Resource | Description |
|----------|-------------|
| [Claude Code — Skills](https://docs.anthropic.com/en/docs/claude-code/skills) | How skills are loaded, where they live, invocation, and how Claude discovers them in Claude Code. |
| [Agent Skills overview](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/overview) | Broader “Agent Skills” model: bundling instructions, scripts, and resources for specialized tasks. |

Pair these with the PDF in **Building skills** and the example `SKILL.md` files in [anthropics/skills](https://github.com/anthropics/skills) when authoring new skills.

## Related: MCP (optional)

Skills define *how* Claude should behave; [MCP](https://modelcontextprotocol.io/) servers add *tools* (browser, DB, docs). Many workflows combine both—e.g. Context7 MCP for docs plus a coding skill for conventions.

| Resource | Description |
|----------|-------------|
| [MCP specification](https://spec.modelcontextprotocol.io/) | Open protocol for connecting models to data and actions outside the chat window. |
| [Claude Code — MCP](https://docs.anthropic.com/en/docs/claude-code/mcp) | Connect Claude Code to external tools: config scopes, transports, and security expectations. |
