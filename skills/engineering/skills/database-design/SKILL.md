---
name: database-design
description: Design database schemas, choose datastores, plan migrations and indexing. Use when the user asks about data models, schema design, SQL vs NoSQL, slow queries, indexes, migrations, or before any backend that persists data is built.
---

# Database Design

Import ../../agentic-engineering-discipline.md. Read prd.md — model the QUERIES the flows need, not just the data.

## Store selection (when green-field)
Default: relational (Postgres-class) unless a specific access pattern demands otherwise (document store for truly schemaless payloads; KV/cache for hot ephemeral data; search engine for full-text). State the decision + rationale. "Web-scale" is not a reason.

## Schema procedure
1. Entities + relationships (text ERD) from the PRD flows.
2. For EACH screen/flow, write the queries it will run. A table nobody queries is dead weight; a query with no supporting shape is a future outage.
3. Normalize to 3NF by default; denormalize ONLY with a named, measured read-pattern justification — document it as a comment in the schema.
4. Types: money as integer minor-units or decimal (never float); timestamps in UTC with timezone type; enums constrained (check/enum); soft-delete only if the product truly needs undelete — otherwise real deletes + audit table.
5. Every table: primary key, created_at/updated_at, FKs with explicit ON DELETE behavior (chosen, not defaulted).

## Indexing rules
- Index FKs and every column in frequent WHERE/ORDER BY — derived from the step-2 query list, not guessed.
- Composite indexes: match column order to query patterns (equality columns first, then range).
- Don't index everything: each index taxes writes. Justify each.

## Migration discipline
- Migrations are code-reviewed, reversible where possible, and NEVER edit an applied migration.
- Zero-downtime pattern for prod changes: add-nullable → backfill → constrain → drop-old (expand/contract).
- Seed data separate from schema migrations.

## Quality gates
- Show the schema DDL + the query list side-by-side; run EXPLAIN on the top queries when an environment exists.
