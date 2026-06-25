# CLAUDE.md — Project Working Agreement

This file instructs Claude Code (and any AI assistant) how to work in this repository.
It is intentionally part of the deliverable: the assessment evaluates **how AI is used**,
not just the final artifact.

## Project

An API test suite for the public [DummyJSON](https://dummyjson.com) API, authored as a
**Postman collection** and runnable headlessly via **Newman** in CI (GitHub Actions).

Coverage targets:

- **Authentication** — JWT issuance, authenticated requests, token refresh, and failure modes.
- **CRUD** — create / read / update / delete on `/products`.
- **Search & filtering** — keyword search, category filtering, pagination, sorting.
- **Error handling** — invalid IDs, missing fields, bad credentials, malformed tokens.

## MANDATORY: Prompt logging

Every prompt the user sends **must** be appended to `prompts.txt` as it happens. This is a
hard requirement of the assessment and must never be skipped.

Format for each entry:

```
## [<ISO 8601 timestamp, e.g. 2026-06-24T22:15:03Z>]
PROMPT:
<the user's prompt, verbatim>

RESPONSE SUMMARY:
<one short paragraph: what was done and why>

---
```

Rules:
- Append-only. Never edit or delete prior entries.
- Log the prompt **before** acting on it where practical, then fill in the response summary.
- Use UTC ISO 8601 timestamps (`YYYY-MM-DDTHH:MM:SSZ`).

## Conventions

- **Ground assertions in real behavior.** Probe the live API before writing tests; do not
  assume the docs are complete or correct. Document any discrepancy found.
- **DummyJSON write operations are simulated** (not persisted). Tests must assert on the
  documented simulated response shape, not on later re-fetching created/updated/deleted data.
- Test scripts use `pm.test` with clear, behavior-describing names.
- Keep secrets out of the repo. The test user (`emilys`) is public demo data from DummyJSON.
- The collection must remain runnable by `newman` with zero manual steps.

## Commands

- `npm test` — run the full suite against the live API via Newman.
- `npm run test:html` — run and emit an HTML report to `newman/`.
