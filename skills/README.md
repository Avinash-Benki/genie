# Claude Skills — Investing + Engineering
Drop this folder into your Cowork workspace. Two domains:

- investing/ — 5 skills + investor-context.md (living state) + lessons.md (append-only). Start any investing session by letting skills read investor-context.md.
- engineering/ — 7 skills + agentic-engineering-discipline.md (shared foundation every skill imports).

Conventions: each skill = skills/<name>/SKILL.md with YAML frontmatter (name, description = trigger). Skills instruct models to SEARCH for live data (prices, NAVs, tax rates, pricing) rather than trusting training data. The context-journal skill keeps investor-context.md current — end sessions with "update the journal".
