# Loom Video Script (~5 minutes)

A talk-track for the demo. Speak naturally — these are beats, not a teleprompter. Times are
rough targets. Have the repo, the Postman app (or a terminal), and the GitHub Actions tab open.

---

## 0:00 — Intro (20s)
"Hi, I'm [name]. This is my submission for the QA Engineer technical assessment — an API test
suite for the DummyJSON API, built as a Postman collection that also runs headlessly in CI.
I'll walk through what it covers, the key decisions I made, and how I worked with AI to build it."

## 0:20 — The shape of the suite (40s)
> Show the four folders in Postman / the collection file.

"It's organized into four folders matching the brief: Authentication, Products CRUD, Search and
Filtering, and Error Handling. In total it's 21 requests and 75 assertions, and they all pass
against the live API. Each request has named `pm.test` assertions that describe behavior, so a
failure reads like a sentence, not a stack trace."

## 1:00 — Decision 1: grounding in real behavior (45s)
"The first thing I did — before writing any tests — was probe the live API to confirm how it
actually behaves: exact status codes, message text, response shapes. I don't trust docs to be
complete, and that paid off: I found a couple of behaviors the docs don't mention. So every
assertion here is grounded in something I observed, not assumed."

## 1:45 — Decision 2: simulated writes (50s)
> Show the CRUD folder, maybe the create test.

"DummyJSON is a mock — it simulates writes but doesn't persist them. If you create a product it
gives you an id, but fetching that id returns a 404. So a naive 'create then read it back' test
would fail forever and look like *my* bug. Instead I assert on the documented simulated response
— the generated id, the echoed fields, the `isDeleted` and `deletedOn` flags on delete — and I
documented that limitation in the README. The docs don't tell you how to handle this, so I made
a defensible call and wrote down my reasoning."

## 2:35 — Decision 3: the cookie-jar bug (55s)
> This is the strongest story — slow down here.

"Here's my favorite finding. My first run had two failures: hitting `/auth/me` with *no* token
returned 200 instead of 401. Turns out DummyJSON sets the JWT as an HttpOnly cookie on login,
and Newman's cookie jar replays it automatically — so my 'unauthenticated' request was secretly
authenticated by a leftover cookie. That's a false pass hiding a broken test. I fixed it with a
pre-request script that clears the cookie jar so the negative auth tests validate the header
contract in isolation. This is the kind of thing you only catch by actually running the suite
end to end — which is why I never just author tests and assume they're correct."

## 3:30 — Defects found (35s)
> Show the README defects table.

"Surfacing issues is the whole point, so I treat findings as first-class output. Top of the list:
sending a malformed token returns a 500 instead of a 401 — a server error where there should be
a clean auth rejection. I assert the actual 500 so the suite stays stable, but flag it clearly
as a known defect. Tests should encode reality and document the bug, not pretend the bug is the
spec."

## 4:05 — Automation & CI (35s)
> Show GitHub Actions green check + package.json scripts.

"The same collection runs three ways: the Postman GUI, the Newman CLI with `npm test`, and
GitHub Actions on every push. CI produces a JUnit report and uploads the results as an artifact.
For a QA role the important signal is repeatable, automated testing — not a collection that only
works on my machine."

## 4:40 — AI workflow (20s)
> Show CLAUDE.md and prompts.txt.

"On the AI side: `CLAUDE.md` is the working agreement, including a rule that logs every prompt to
`prompts.txt` with timestamps. My approach was AI for speed, me for judgment and verification —
I directed the strategy and confirmed every behavior against the live API before committing it."

## 5:00 — Close (10s)
"That's the suite — grounded in real behavior, automated end to end, and honest about what it
found. Thanks for watching; the repo link is in the submission."

---

### Pre-record checklist
- [ ] `npm test` shows all green (record this live if possible — it's convincing)
- [ ] GitHub Actions run is green and visible
- [ ] Postman app open with the four folders expanded
- [ ] README defects table and prompts.txt visible
- [ ] Mic check; close noisy apps/notifications
