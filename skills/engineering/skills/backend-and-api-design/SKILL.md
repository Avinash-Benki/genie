---
name: backend-and-api-design
description: Design and build backend services and APIs with security baked in. Use when the user asks for an API, server, backend logic, authentication, integration, webhooks, background jobs, or reviews backend code. Framework-neutral.
---

# Backend & API Design

Import ../../agentic-engineering-discipline.md. Read prd.md; the API serves its flows — nothing more (minimal-code).

## API-first rule
Write the contract BEFORE handlers: resources, endpoints, request/response shapes, error format, status codes. Default to a consistent error envelope (code, message, details). Version from day one (/v1/).

## Design rules
- Nouns for resources, verbs via HTTP methods; plural collections; nested only one level deep.
- Validation AT THE BOUNDARY: every input validated before business logic; whitelist fields explicitly (mass-assignment guard).
- Pagination on every list endpoint from day one (cursor preferred over offset for growing data).
- Idempotency for anything payment/creation-critical (idempotency keys).
- Async for slow work: request → job → status endpoint/webhook; never block a request on >2s work.

## Security checklist (LLMs notoriously skip these — verify each explicitly)
1. AuthN vs AuthZ separated; every endpoint declares required role/ownership. IDOR check: can user A fetch user B's resource by changing an ID?
2. Parameterized queries ONLY (injection guard) — string-built SQL is a defect regardless of "sanitization".
3. Secrets in env/secret manager; never in code, logs, or error responses.
4. Rate limiting on auth + expensive endpoints; lockout/backoff on login.
5. Password hashing: modern KDF (argon2/bcrypt-class); never roll your own crypto.
6. CORS: explicit origins, never * with credentials.
7. Error responses leak nothing internal (stack traces, SQL, paths) in production.

## Quality gates
- Contract documented; happy path + at least the top 3 error paths tested; logs structured with request IDs; health endpoint exists.
- If it hasn't run against a real request, it doesn't exist.
