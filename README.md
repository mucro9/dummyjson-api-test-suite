# DummyJSON API Test Suite

A professional API test suite for the public [DummyJSON](https://dummyjson.com) API, authored
as a **Postman collection** and runnable headlessly via **Newman** in **GitHub Actions** CI.

[![API Tests](https://github.com/mucro9/dummyjson-api-test-suite/actions/workflows/api-tests.yml/badge.svg)](https://github.com/mucro9/dummyjson-api-test-suite/actions/workflows/api-tests.yml)

---

## What this covers

| Area | Scenarios |
|------|-----------|
| **Authentication** | Login + JWT issuance, well-formed-JWT check, authenticated `/auth/me`, token refresh, invalid credentials, missing field, no token, malformed token |
| **CRUD** | Create, read, update (PUT), delete on `/products` |
| **Search / Filter / Pagination** | Keyword search, no-result search, category filter, category list, `limit`/`skip` pagination, field selection, sorting |
| **Error handling** | Non-existent id (404), non-numeric id, unknown category, validation errors, auth failures |

**21 requests · 75 assertions · all green** against the live API.

---

## Quick start

```bash
npm install        # installs Newman + reporters
npm test           # runs the suite against the live DummyJSON API
npm run test:html  # runs and writes an HTML report to newman/report.html
```

You can also import `collections/dummyjson.postman_collection.json` and
`environments/dummyjson.postman_environment.json` directly into the Postman app and hit **Run**.

---

## How I approached this (decisions & trade-offs)

This was an AI-assisted build with Claude Code. My role was to **direct the strategy, verify
every claim against reality, and make the judgment calls**; the AI accelerated the mechanical
work. The decisions below are the substance of the suite.

### 1. I grounded every assertion in real API behavior before writing a single test
Rather than trusting the published docs, I probed the live endpoints first (login, CRUD,
search, error cases) and wrote assertions against **observed** responses — exact status codes,
message text, and response shapes. API docs are frequently incomplete or stale; tests built on
assumptions are tests that lie. This also surfaced two behaviors the docs don't mention (below).

### 2. DummyJSON simulates writes — so I tested the contract, not persistence
`POST /products/add`, `PUT /products/:id`, and `DELETE /products/:id` all return realistic
responses but **do not persist**. A created product (`id: 195`) returns `404` when re-fetched;
delete returns `isDeleted: true` rather than removing the record.

A naive "create then GET it back" test would fail forever and look like a bug in my suite. The
right call for a mock API is to assert on the **documented simulated response** — generated id,
echoed fields, `isDeleted`/`deletedOn` flags — and to document the persistence limitation
explicitly. This is the kind of ambiguity the assessment asks about: the docs don't cover it,
so I made a defensible decision and wrote it down.

### 3. I handled a real test-isolation trap: the HttpOnly auth cookie
My first run had two failing assertions: `GET /auth/me` with **no** `Authorization` header
returned `200` instead of `401`. Root cause: DummyJSON's login response sets the JWT as an
**HttpOnly cookie**, and Newman's cookie jar automatically replays it on subsequent requests —
so the "unauthenticated" request was silently authenticated by a leftover cookie.

Fix: a pre-request script clears the cookie jar before the negative-auth tests, so they
validate the `Authorization`-header contract in isolation. This is a classic false-negative
(here, a false *pass* hiding a broken test) that only shows up when you actually run the suite
end-to-end — which is exactly why I insisted on running it, not just authoring it.

### 4. I built for automation, not just for clicking
The suite runs identically in three places: the Postman GUI, the `newman` CLI, and GitHub
Actions on every push/PR. CI emits a JUnit XML report (consumable by dashboards) and uploads
results as an artifact. For a QA role, the signal that matters is *repeatable, automated*
testing — not a collection that only works on one laptop.

### 5. Stability choices to avoid flaky tests
- A generous global 5s response-time assertion (it's a public demo API; tight thresholds flake).
- Keyword-search assertions check the term across all searchable fields (title, description,
  category, tags), because DummyJSON matches more broadly than title-only.
- Tokens are captured into environment variables and chained between requests, so the auth flow
  is realistic and self-contained.

---

## 🐞 Defects / observations found

Surfacing issues is the point of testing, so these are first-class output, not footnotes:

| # | Observation | Expected | Actual | Where |
|---|-------------|----------|--------|-------|
| 1 | A **malformed bearer token** is sent to `/auth/me` | `401 Unauthorized` | **`500` Internal Server Error** | Auth folder — asserted and labelled `[KNOWN DEFECT]` |
| 2 | Login sets the JWT as an **HttpOnly cookie**, which the docs don't mention | Header-based auth only (per docs) | Cookie also authenticates; can mask missing-header bugs | Handled via cookie-jar isolation |
| 3 | Write operations (`add`/update/`delete`) are **not persisted** | Stated as a limitation, but easy to miss | Simulated responses only | Tests assert the simulated contract |

For (1) I assert the *actual* `500` so the suite stays green and stable, while clearly flagging
the gap — a test suite should encode reality and document the bug, not pretend the bug is the
spec.

---

## Project structure

```
.
├── collections/
│   └── dummyjson.postman_collection.json   # the test suite (4 folders)
├── environments/
│   └── dummyjson.postman_environment.json  # baseUrl, demo creds, token vars
├── .github/workflows/
│   └── api-tests.yml                        # runs Newman on push/PR
├── CLAUDE.md                                # AI working agreement + prompt-logging rule
├── prompts.txt                              # append-only log of every prompt (per assessment)
├── package.json                             # npm test / test:html / test:ci
└── README.md
```

---

## AI workflow & transparency

Per the assessment, `CLAUDE.md` defines how the AI assistant works in this repo, including a
mandatory rule to log **every prompt** to `prompts.txt` with ISO 8601 timestamps and a response
summary. `prompts.txt` is the honest, append-only record of how this suite was built.

The principle throughout: **AI for speed, human for judgment and verification.** Every behavior
the suite asserts was confirmed against the live API before being committed.
