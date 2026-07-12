---
name: deployment-and-devops
description: Ship and operate applications — environments, CI/CD, containers, monitoring, rollback. Use when the user asks to deploy, host, dockerize, set up CI, "put it live", choose hosting, handle env vars/secrets, or when an app is ready for users.
---

# Deployment & DevOps

Import ../../agentic-engineering-discipline.md. Match effort to the declared tier (prototype/internal/production) — do not gold-plate a demo, do not YOLO production.

## Environment strategy
Minimum two (local, prod); add staging when real users exist. Parity via containers or lockfiles + pinned runtimes. All config via environment variables; a committed .env.example documents every variable (values blank).

## CI/CD baseline (any stack)
Pipeline stages in order: install → lint → test → build → deploy. A red stage blocks deploy — no manual overrides normalized. Deploy previews per branch/PR where the platform supports it.

## Hosting selection
Choose by tier and shape, not fashion: static/frontend → CDN-backed static hosts; simple full-stack → PaaS; containers when you need runtime control; managed DB over self-hosted at MVP scale. State monthly cost estimate for the chosen option (search current pricing).

## Pre-deploy checklist (production tier — verify each)
1. Secrets in the platform's secret store; none in the repo history (scan).
2. DB migrations applied via pipeline, not by hand.
3. Error tracking wired (any APM/error service) + structured logs retained.
4. Health endpoint + uptime monitor + alert to a channel the user actually reads.
5. Backups: DB automated daily minimum; restore tested ONCE (an untested backup is a hope, not a backup).
6. Rollback plan written: exactly how to revert app AND schema (expand/contract makes schema rollback possible).
7. Domain, TLS, security headers (HSTS, CSP baseline), robots/analytics as intended.

## Operating rules
- Deploy small and often; every deploy maps to a commit range.
- Post-incident: 5-line blameless note (what broke, why, detection gap, fix, prevention) appended to the project's ops.md.
