# Loom Video Script (~5 minutes)

A talk-track for the demo. Speak naturally — these are beats, not a teleprompter. Times are
rough targets.

> **Per the recruiter (Andres):** run the collection **live in the Postman app** and walk
> through it — show the requests executing and the test results (assertions passing/failing)
> in the **Postman Collection Runner**. Newman is *not* needed for the demo. The exported
> collection JSON + environment file is the required deliverable; Newman + GitHub Actions is a
> welcome *bonus* to show range. **Primary evaluation is the quality of the test design** — so
> spend your time on the decisions, not the tooling.

**Have open before recording:** the Postman app with the collection imported and the `DummyJSON
— Live` environment selected; the GitHub repo (README defects table + prompts.txt); optionally
the green GitHub Actions tab for the closing bonus beat.

---

## 0:00 — Intro (20s)
"Hi, I'm [name]. This is my submission for the QA Engineer technical assessment — an API test
suite for the DummyJSON API, built as a Postman collection. I'll run it live in Postman, walk
through the key test-design decisions I made, and show how I worked with AI to build it."

## 0:20 — Run it live in Postman (50s)
> Open the **Collection Runner**, select the `DummyJSON — Live` environment, hit **Run**. Let
> it execute on camera. Then expand a few requests to show the named assertions ticking green.

"First, let's just run it. I'll select the live environment and kick off the Collection Runner…
21 requests, 75 assertions, all green against the live API. The point I want to land: each
assertion has a behavior-describing name — so when something fails it reads like a sentence, not
a stack trace. Let me expand a couple so you can see the test results inline."

## 1:10 — The shape of the suite (30s)
> Show the four folders in the collection sidebar.

"It's organized into four folders matching the brief: Authentication, Products CRUD, Search and
Filtering, and Error Handling. Now I'll walk through the design decisions that I think matter
most — because that's really what this suite is about."

## 1:40 — Decision 1: grounding in real behavior (40s)
"Before writing a single assertion, I probed the live API to confirm how it *actually* behaves —
exact status codes, message text, response shapes. I don't trust docs to be complete, and that
paid off: I found behaviors the docs don't mention. So every assertion here is grounded in
something I observed, not something I assumed."

## 2:20 — Decision 2: simulated writes (50s)
> Open the CRUD folder; show the create/update/delete tests and their assertions in the runner.

"DummyJSON is a mock — it simulates writes but doesn't persist them. Create a product and it
returns an id, but fetching that id later gives a 404. A naive 'create then read it back' test
would fail forever and look like *my* bug. So instead I assert on the documented simulated
response — the generated id, the echoed fields, the `isDeleted` and `deletedOn` flags on delete
— and I documented that limitation in the README. The docs don't tell you how to handle this; I
made a defensible call and wrote down the reasoning."

## 3:10 — Decision 3: the cookie-jar false pass (55s)
> Strongest story — slow down. Show the negative-auth test and its pre-request script.

"Here's my favorite finding. An early run had a request to `/auth/me` with *no* token returning
200 instead of 401. Turns out DummyJSON sets the JWT as an HttpOnly cookie on login, and the
cookie jar replays it automatically — so my 'unauthenticated' request was secretly authenticated
by a leftover cookie. That's a false *pass* hiding a broken test. I fixed it with a pre-request
script that clears the cookie jar, so the negative auth tests validate the header contract in
isolation. You only catch this by actually running the suite — which is exactly why I never just
author tests and assume they're correct."

## 4:05 — Defects surfaced (35s)
> Show the README defects table.

"Surfacing issues is the whole point, so I treat findings as first-class output. Top of the list:
a malformed token returns a 500 instead of a 401 — a server error where there should be a clean
auth rejection. I assert the actual 500 so the suite stays stable, but I flag it clearly as a
known defect. Tests should encode reality and document the bug — not pretend the bug is the spec."

## 4:40 — AI workflow + bonus (40s)
> Show CLAUDE.md and prompts.txt. Optionally flash the green GitHub Actions tab.

"On the AI side: `CLAUDE.md` is my working agreement, including a rule that logs every prompt to
`prompts.txt` with timestamps. My approach was AI for speed, me for judgment — I directed the
strategy and verified every behavior against the live API before committing. And as a bonus to
show range, the same collection also runs headlessly via Newman in GitHub Actions on every push,
with a JUnit report — but the heart of the submission is the test design you just saw."

## 5:20 — Close (10s)
"That's the suite — grounded in real behavior, honest about what it found, and runnable by anyone
who imports it. Thanks for watching; the repo link is in the submission."

---

### Pre-record checklist
- [ ] Collection imported into the Postman app; `DummyJSON — Live` environment selected
- [ ] Collection Runner does a clean all-green pass (do a dry run first)
- [ ] Four folders expanded in the sidebar; a couple of test scripts ready to show
- [ ] README defects table and prompts.txt visible in the repo
- [ ] (Bonus) GitHub Actions green check in a tab
- [ ] Mic check; close noisy apps/notifications
