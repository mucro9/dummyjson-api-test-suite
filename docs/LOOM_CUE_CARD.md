# Loom Cue Card — read-aloud script (~5 min)

Just read the **bold-free lines**. Text in `[brackets]` is a stage cue — do the action, don't say it.
Pause briefly at each `[click ...]`. Speak a little slower than feels natural; it always sounds rushed on playback.

Replace `[NAME]` with your name in the first line.

---

### 1. Intro  (~20s)
`[Screen on the repo README or Postman app]`

"Hi, I'm [NAME]. This is my submission for the QA Engineer technical assessment — an API test suite for the public DummyJSON API, built as a Postman collection. I'll run it live in Postman, walk through the main test-design decisions I made, and show how I worked with AI to build it."

---

### 2. Run it live  (~50s)
`[Open the Collection Runner — left sidebar, hover "DummyJSON API Test Suite", click ••• → Run collection]`

"Let me start by just running the whole thing. I've got the live environment selected, and I'll kick off the Collection Runner."

`[Click Run. Let it finish.]`

"There it goes — twenty-one requests, seventy-five assertions, all green against the live API."

`[Expand one or two requests to show the named assertions]`

"The detail I care about: every assertion has a behavior-describing name. So when something fails, it reads like a sentence — 'status code is 401', 'explains that a token is required' — not a stack trace. That makes failures self-documenting."

---

### 3. The four folders  (~25s)
`[Point at the four folders in the sidebar]`

"The suite is organized into four folders that match the brief: Authentication, Products CRUD, Search and Filtering, and Error Handling. Now I want to walk through the three decisions that I think actually matter — because that's what this suite is really about."

---

### 4. Decision 1 — ground everything in real behavior  (~40s)

"First: before I wrote a single assertion, I probed the live API to confirm how it actually behaves — exact status codes, the message text, the response shapes. I don't trust documentation to be complete or current. And that paid off — I found a couple of behaviors the docs don't mention, which I'll show you. So every assertion here is grounded in something I observed, not something I assumed."

---

### 5. Decision 2 — simulated writes  (~50s)
`[Open the Products — CRUD folder; show the create/update/delete tests]`

"Second: DummyJSON is a mock API. It simulates writes but doesn't persist them. So if I create a product, it returns a new id — but if I then fetch that id, I get a 404. A naive 'create it, then read it back' test would fail forever and look like a bug in my suite."

"So I made a deliberate call: I assert on the documented simulated response — the generated id, the echoed fields, the isDeleted and deletedOn flags on delete — and I documented that limitation in the README. The docs don't tell you how to handle this, so I made a defensible decision and wrote down my reasoning."

---

### 6. Decision 3 — the cookie-jar false pass  (~60s)  ← your strongest story, slow down
`[Open the Authentication folder; click the "Get current user — no token (401)" request and show its pre-request script]`

"Third, and this is my favorite finding. One of my negative tests hits /auth/me with no token, and it should return 401. But on an early run it returned 200 — a pass that was actually wrong."

"The cause: when you log in, DummyJSON sets the JWT as an HttpOnly cookie. The cookie jar then replays that cookie automatically — so my 'unauthenticated' request was secretly authenticated by a leftover cookie. That's a false pass hiding a broken test, which is one of the more dangerous things in testing."

"I fixed it with a pre-request script that clears the cookie jar, so the negative-auth test validates the Authorization-header contract in isolation. And here's a nice detail — this behaves differently across runners: in Newman it just works, but the Postman desktop app sandboxes cookie access, so I had to allowlist the domain for the script to run here too. I documented both. You only catch this kind of thing by actually running the suite end-to-end — which is exactly why I never just author tests and assume they're correct."

---

### 7. Defects surfaced  (~35s)
`[Switch to the repo README; scroll to the Defects table]`

"Surfacing issues is the whole point of testing, so I treat findings as first-class output. The headline one: sending a malformed token to /auth/me returns a 500 — a server error — where it should return a clean 401. I assert the actual 500 so the suite stays stable and green, but I flag it clearly as a known defect. A test suite should encode reality and document the bug, not pretend the bug is the spec."

---

### 8. AI workflow + bonus  (~40s)
`[Show CLAUDE.md and prompts.txt in the repo. Optionally flash the green GitHub Actions tab.]`

"On the AI side: I set up a working agreement in CLAUDE.md, including a rule that logs every prompt I send to a prompts.txt file with timestamps — so there's an honest, append-only record of how this was built. My approach throughout was AI for speed, me for judgment: I directed the strategy and verified every behavior against the live API before committing it."

"And as a bonus to show range, the same collection also runs headlessly in Newman through GitHub Actions on every push, with a JUnit report — but the heart of the submission is the test design I just walked you through."

---

### 9. Close  (~10s)

"That's the suite — grounded in real behavior, honest about what it found, and runnable by anyone who imports it. The repo link is in the submission. Thanks for watching."

---

## Quick reminders before you hit record
- Do one silent dry-run of the Runner first so it's a clean 75/75 on camera.
- Close the AI panel on the right in Postman; expand the 4 folders.
- Have two windows ready: Postman, and the browser on your GitHub repo (README + prompts.txt).
- Speak slightly slower than feels natural. One take doesn't have to be perfect — Loom lets you re-record.
