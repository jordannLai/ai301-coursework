# Contribution: Implement Jest Tests for Badge Maker for Readme

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Jordan Lai
**GitHub Username:** jordannLai
**Issue:** https://github.com/Bashamega/WebDevTools/issues/400
**Pull Request:** https://github.com/Bashamega/WebDevTools/pull/704

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: Ran issue-scout's first live GitHub scan, then graded hand-picked candidates until one passed (2026-10-19)**

I ran my issue-scout skill live: three `good first issue` searches (TypeScript, Python, Java) drawn from my fit profile, a ranking script that dropped assigned, stale, and noisy issues, and rubric grading of the top five. The scan came back 0/5 accepted, with every pick failing only Issue availability: my ranking checked assignees but couldn't see open linked PRs or claim comments, sorting by most recent update pushed freshly claimed issues to the top, and Query C hit the 100-result limit, so it may have been truncated. I logged these limits in `scan-notes.md` along with the strategy fixes they call for (drop issues with open linked PRs or recent claims, stop favouring recent updates, split Query C). Then I graded hand-picked candidates directly; WebDevTools #400 passed every required check, and I chose it because it is small, concrete, test-focused JavaScript/React work.

**Unit 6: Installed the voice guard, patched the scout's scope check, and wrote my WebDevTools repo memory (2026-11-15)**

I installed the voice guard: `check_voice.py` is a PreToolUse hook in `~/.claude/settings.json`, and my voice guide is promoted to `~/.claude/voice-guard/voice-guide.md`, with repro-check, plan-check, and pr-precheck symlinked to it. I added an Upstream section to the guide (claims, questions, pings). The wiring test (`gh issue comment --help`) was stopped with "no body the guard can read", which is the intended pass. My claim follow-up passed both the manual `--draft` check and the hook when I posted it (`guard-log.md` has all three lines). One thing fell short: an example rule I first wrote said the badge API returns status 200 on missing input. Reading `route.js` showed it returns 500, so I fixed the example before relying on it. I also replaced my rubric's "Newcomer scope" check with the Unit 6 `scope-feasible` row plus a difficulty note, and wrote `CLAUDE.local.md` in my clone (kept out of git through `.git/info/exclude`) so every session starts with the repo's rules: tests mirror `src/app`, CodeRabbit gates review, the link-count auto-moderator, and `npm run lint` already broken on main.

**Unit 7: Installed the ship-gate, wrote my WebDevTools manifest, and shipped PR #704 through it (2026-11-15)**

`install.py --repo ~/Desktop/ai301/WebDevTools` added the hook beside my voice guard (with a dated settings backup), swapped pr-precheck to the live scope, and created a git-excluded `ship-manifest.md`. I filled it with the three checks the repo actually runs: `npm run format:check` (format.yml and Husky), `npm test` (contributing.md, jest_tests.yml, Husky) and `npm run test:coverage` (jest_tests.yml). `npm run lint` went in the dated baseline-failure table, because it already fails on clean main (Next 16 removed `next lint`). The hook test stopped twice, once from each hook. On my real branch the gate passed 3/3 checks and pr-precheck accepted (7/7 required) on both runs. Two things fought me. Prettier's `format:check` scans the whole clone, so my own untracked manifest failed the gate's first check until I formatted it. And the hook is user-global, so it also stopped a local editing command whose heredoc mentioned the PR command. What changed: I format every local working file, I fixed a PR line pr-precheck's voice notes caught (I'd attributed the maintainer's "each fix in a separate pr" to #400; it was said on #391), and I left the rubric alone, because it didn't ask for anything this repo doesn't.

**Unit 8: Installed the PR monitor, built my playbook from WebDevTools' real review rhythm, and let it tell me to wait (2026-11-15)**

I installed the monitor with the three commands. There was no existing `playbook.md`, so nothing was overwritten. `monitor.py --state` on my branch found PR #704. I measured the playbook's thresholds from five merged outside-contributor PRs (#702, #701, #697, #696, #693; bots and the maintainer skipped), where Bashamega's first review came after 4.2–15.8h. So the first-review nudge waits 4 quiet days, and a stalled re-review waits 5. The response rules treat CodeRabbit comments as review, because the maintainer requested changes until they were applied on #693, #696 and #701. What fell short: the monitor labeled the PR "FEEDBACK-IN" because of two bot comments (Vercel's deploy authorization, and CodeRabbit's "No actionable comments were generated"), and it couldn't name the one failing check. Its drafted move, following my playbook, was a wait note: no post, earliest nudge about 2026-11-20. I didn't post anything, I confirmed the failing check is the Vercel authorization gate that merged outside PRs also show, and I wrote no `my-reply.md`, because there's no feedback to answer.

---

## Why I Chose This Issue

I started with a live issue-scout scan on 2026-10-19: three `good first issue` searches (TypeScript, Python, and Java owners taken from my fit profile), a ranking pass that dropped assigned, stale, noisy, and umbrella issues, and rubric grading of the top five. It accepted 0 of 5, and all five failed only Issue availability. My ranking could see assignees but not open linked PRs or claim comments, so it kept surfacing issues someone was already working on (iceberg, sequelize, typeorm, sedona, gravitino all had open linked PRs or fresh claims). Sorting by most recently updated made it worse, because on a "good first issue" a recent update usually means a new claim. Query C also returned exactly 100 results, the `--limit`, so it may have been truncated. I then graded AI301 Issue Selection site picks directly in four batches. The first two accepted nothing, the third accepted only a weak docs issue (rawtoaces#100), and the fourth accepted all three WebDevTools picks, with #400 ranked first. In Unit 6 I patched the rubric (below) and re-graded #400 on 2026-11-15. It's still **accept**, with difficulty note **moderate**: the issue names the tool but not the files, and three files are involved (`BadgeForm.jsx`, `page.jsx`, `api/badge/route.js`).

#400 fits me because it's testing work on a real React/Next.js codebase that I can run end to end locally (87/87 tests and the build pass on main), and it includes a server route, which matches the "writing tests, improving reliability" and full-stack parts of my profile. It's one bounded module, and the maintainer asks for one module per PR. By November the October risk had also gone away: Bashamega merged six outside PRs between 2026-11-05 and 2026-11-12, four of them Jest-test PRs for other tools. I want to learn RTL testing of client components, testing a Next.js route handler with Web `Request`/`Response`, and how this maintainer's CodeRabbit-gated review works. My honest read of the scout: its rubric grading held up (each rejection in `scan-output.md` names a specific open PR or stranger claim, with dates), but its ranking was blind to availability, and that cost me a whole scan. The scope check was too strict for the wild, so this week I replaced it.

Claim comment: https://github.com/Bashamega/WebDevTools/issues/400#issuecomment-6273810106 (2026-10-19). The maintainer replied "Feel free to work on it" the same day. My claim didn't say when I'd report back, and it put every test under `__tests__/badge-maker/`, which conflicts with contributing.md, so I posted a voice-guarded follow-up: https://github.com/Bashamega/WebDevTools/issues/400#issuecomment-6907202386 (2026-11-15).

**Toolkit**

- Tool: issue-scout. A live scan with my `search-strategy.md` queries, then direct-mode grading of site picks against my `rubric.md`, then a direct re-grade of #400 with the Unit 6-patched rubric.
- What it produced: The scan accepted 0/5 (all failed Issue availability). Direct batches accepted 0/3, 0/4, 1/3, then 3/3 for WebDevTools with #400 first. The 2026-11-15 re-grade said: accept, all 5 required checks pass, difficulty "moderate". Its remaining risks were the test-path rule vs. my claim, the route returning 500 on missing input, CodeRabbit-gated review, the auto-moderator's link limit, and `npm run lint` broken on main.
- Difficulties: The ranking couldn't see claims or open linked PRs (the availability blind spot), so the whole scan was spent on taken issues. Recency sorting favored freshly claimed issues. Query C hit the 100-result limit. And the old "Newcomer scope" row (reject on unresolved design decisions, "core internals", "too complex"), plus a stricter manual rule I added in batch 3, would reject most honest wild issues that have an open edge.
- What changed: In Unit 6 I replaced the "Newcomer scope" row with `scope-feasible`, which fails only issues that aren't one piece of work and turns open edges into a straightforward/moderate/ambitious note. It's logged in `scan-notes.md` (2026-11-15). Still to do from `scan-notes.md`: drop issues with open linked PRs or recent claims during ranking, stop sorting by recent update, and split Query C.

---

## Understanding the Issue

**Problem Description**

Nothing in the Badge Maker for Readme is broken. What's missing is test coverage: no Jest test exercises it, so a change to the form, the page, or the `/api/badge` route could break it and CI would still pass. The issue asks for unit tests (component rendering, user interactions, props and state), integration tests for significant workflows if applicable, everything passing locally and in CI, and docs on how to run the tests.

**Expected Behavior**

Jest/React Testing Library tests for the Badge Maker exist and pass, locally and in the "Run Jest Tests" workflow. Between them they should check that the form renders its fields with the right defaults, that typing updates the fields, that submitting passes the right data up to the page, that the page turns that into a preview URL and a Markdown snippet (copy, theme), and that the API returns an SVG badge for valid input and an error for missing input. Per contributing.md, the tests live in `__tests__/` mirroring the `src/app` path. The "how to run the tests" docs part already exists on main.

**Current Behavior**

On `main` at `c9cdcc3`, Jest collects 13 suites (87 tests, all passing) and none of them import a Badge Maker file. `npx jest --listTests | grep -i badge` finds nothing, and coverage for `BadgeForm.jsx`, `page.jsx` and `route.js` is 0% across statements, branches, functions and lines. At runtime the feature works: the route returns 200 `image/svg+xml` for `?label=build&message=passing`, and 500 `application/json` `{"error":"Label and message are required"}` when label or message is missing.

**Affected Components**

- `src/app/badge-maker/BadgeForm.jsx` (177 lines): a client form with seven controlled fields. On submit it calls the `onSubmit` prop with `{label, message, labelColor, color, logoBase64, links, style}`, where `links` is split per line, trimmed, and stripped of blanks. The `isDarkMode` prop switches the background.
- `src/app/badge-maker/page.jsx` (96 lines): renders `NavBar` ("Badge Maker"), `BadgeForm` and `Footer`. `handleGenerateBadge` builds `${window.location.origin}/api/badge?${new URLSearchParams(data)}` for the preview `<img>` and the `![Badge](url)` Markdown. `copyToClipboard` uses `navigator.clipboard` then `alert`. The theme toggle saves to and restores from localStorage `theme`.
- `src/app/api/badge/route.js` (43 lines): `GET` reads the query params, calls `makeBadge()` from `badge-maker` 6.0.0, and returns the SVG, or a 500 JSON error.
- The test setup it plugs into: `jest.config.js` (next/jest, jsdom, `@/` maps to `src/`), `jest.setup.js` (jest-dom), `.github/workflows/jest_tests.yml`.

**Toolkit**

- Tool: Claude Code in my WebDevTools clone, with `CLAUDE.local.md` loaded. I read the three Badge Maker files, NavBar, and the merged Box Shadow and Readme generator tests, and ran a throwaway Jest probe against the route. issue-scout's re-grade framed the difficulty as "moderate".
- What it produced: The behavior map above (each field, the submit payload, the preview URL format, the copy and theme paths, the route's 200 and 500 responses) and the style conventions to copy: real NavBar, `navigator.clipboard` stubbed through `Object.defineProperty`, `localStorage.clear()` in `beforeEach`.
- Difficulties: I first assumed the API returned 200 with an error on missing input. Reading `route.js` and probing it showed a 500. badge-maker 6 is ESM-only, so I expected an import problem in Jest, but the probe showed none under `npm test`. The issue doesn't name any files, so finding them was my job.
- What changed: I fixed the 500/200 example in my voice guide, recorded "test current behavior, don't change the status code" as a scope rule in `CLAUDE.local.md` and the plan, and moved the route test location to `__tests__/api/badge/` to follow contributing.md.

---

## Reproduction Process

#400 is a missing-test-coverage task, not a runtime bug, so there's no bug to trigger. What I reproduced is the state the issue describes: no Jest test covers the Badge Maker. I also recorded what the code does now, so the tests can assert real behavior.

**Environment Setup**

- macOS 26.5.2 (build 25F84), Node v25.2.1, npm 11.6.2.
- Bashamega/WebDevTools `main` at `c9cdcc386d230ddfa5ff17aff169b4d992b00396` (2026-11-12). I forked it to jordannLai/WebDevTools, cloned the fork to `~/Desktop/ai301/WebDevTools`, added `upstream`, and ran `npm ci`.
- Test stack from `package-lock.json`: jest 30.5.1, jest-environment-jsdom 30.4.1, @testing-library/react 16.3.2, @testing-library/jest-dom 7.0.1, next 16.4.5 (`next/jest` config in `jest.config.js`; `@/` maps to `src/`), react 19.2.7, badge-maker 6.0.0.
- CI (`.github/workflows/jest_tests.yml`) runs `npm test` and `npm run test:coverage`. `format.yml` runs `npm run format:check`.
- Challenges: badge-maker 6 is ESM-only (`"type": "module"`), and I expected Jest not to load it. A throwaway test showed it imports fine under `npm test`, which sets `NODE_OPTIONS=--experimental-vm-modules`. The route needs a `@jest-environment node` docblock because it uses `Request`/`Response`. `npm run lint` fails on clean main ("Invalid project directory provided, no such directory: .../lint", since Next 16 removed `next lint`). It isn't run in CI, so I recorded it as a baseline failure.

**Steps to Reproduce**

1. Following contributing.md's Development Workflow: `git clone https://github.com/Bashamega/WebDevTools.git && cd WebDevTools && git checkout c9cdcc3 && npm ci`.
2. `npx jest --listTests | grep -i badge || echo "NO badge tests found"` lists what Jest collects and checks for Badge Maker tests.
3. `npm test` is the baseline suite.
4. `NODE_OPTIONS=--experimental-vm-modules npx jest --coverage --collectCoverageFrom='src/app/badge-maker/**' --collectCoverageFrom='src/app/api/badge/**' --coverageReporters=text` gives Badge Maker coverage on main.
5. A throwaway `@jest-environment node` test calls `GET` from `@/app/api/badge/route` with `?label=build&message=passing` and with only `?label=build`, printing the status, Content-Type and body. I deleted it after the run, and `git status` was clean.
6. Observed: Jest collects 13 suites (7 in `__tests__/`, 6 colocated in `src/app/customizer/`), none for the Badge Maker ("NO badge tests found"). The suite passes at 13/13 suites and 87/87 tests. All three Badge Maker files are at 0% coverage. The route returns 200 `image/svg+xml` with valid input, and 500 `application/json` `{"error":"Label and message are required"}` when the message is missing.

**Reproduction Evidence**

- **Commit showing reproduction:** No reproduction commit, because nothing needed changing to show a missing test. The evidence is the commands above run on unmodified upstream `main` at https://github.com/Bashamega/WebDevTools/commit/c9cdcc386d230ddfa5ff17aff169b4d992b00396.
- **Screenshots/logs:**

```
$ npx jest --listTests | grep -i badge || echo "NO badge tests found"
NO badge tests found

$ npm test
Test Suites: 13 passed, 13 total
Tests:       87 passed, 87 total

$ ... jest --coverage --collectCoverageFrom=src/app/badge-maker/** --collectCoverageFrom=src/app/api/badge/**
----------------|---------|----------|---------|---------|-------------------
All files       |       0 |        0 |       0 |       0 |
  route.js      |       0 |        0 |       0 |       0 | 1-36
  BadgeForm.jsx |       0 |        0 |       0 |       0 | 4-158
  page.jsx      |       0 |        0 |       0 |       0 | 4-96

(throwaway route probe)
OK 200 image/svg+xml <svg xmlns="http://www.w3.org/2000/svg" width="88" height="20" role="img" aria-label="build: passing">...
(console.error) Error generating badge: Label and message are required
BAD 500 application/json {"error":"Label and message are required"}
```

- **My findings:** The gap the issue describes is real and unchanged on current main. The only "badge" match in `__tests__/` is the Readme generator's own "Badges" field. The feature works: valid input returns an SVG, and missing input returns a JSON error. Two things I'm reporting but not changing: a missing label or message returns **500** where a 4xx would be more accurate, and `page.jsx` builds the query with `new URLSearchParams(data)`, which joins the `links` array into one comma-separated value. The issue's "update the documentation" task is already done on main (contributing.md "Run the Tests", from #692). And contributing.md says tests mirror the `src/app` path, which corrected where my claim said the route test would go. I posted that correction in the follow-up comment.

**Toolkit**

- Tool: repro-check, run as `claude "repro-check: grade my claim comment in claim-draft.md and my repro report in repro-draft.md for issue …/400"` from my Unit 6 folder, with the Unit 6 wild scope and my voice guide. Then the voice guard on the follow-up comment (manual `--draft` check, then the hook on `gh issue comment --body-file`).
- What it produced: **accept** on the first run. All six required checks passed: it re-verified the SHA as current main, matched the 500 behavior to `route.js:23-38`, and confirmed no AI policy and no competing claim. It made two voice notes: "within the next two days" promised a PR date (which my claims rule says not to do), and posting the repro and the follow-up separately would repeat the same facts. It also suggested pasting the throwaway test and fixing my fork-vs-upstream clone wording.
- Difficulties: The rubric's proof families assume a bug to reproduce. For a missing-test issue, "reproduction" had to mean evidence of absence (test listing, grep, 0% coverage) plus a record of current behavior, and I had to say so plainly rather than invent a bug. My original claim also didn't fully meet the Unit 6 claim requirements (no report-back time).
- What changed: I reworded the timeline to "I'll report back here within two days with a PR link or a status update", added the probe code to the repro report, and fixed the clone wording. I posted one combined follow-up instead of a separate repro comment. The voice guard passed it (manual and hook).

---

## Solution Approach

**Analysis**

The "root cause" is simply that no test file targets the Badge Maker. Other tools have been getting tests one PR at a time (#692 Readme generator, #696 Image Color Picker, #697 Lorem Ipsum, #701 Box Shadow, #702 AsciiDoc), and the Badge Maker hasn't been done yet. The control run on the route (200 SVG with valid input, 500 JSON on missing input) shows the feature works, so there's no app bug and app code shouldn't change. The behaviors worth locking in are the ones in Affected Components: `BadgeForm`'s controlled fields and the `handleSubmit` payload (especially the `links` split/trim/filter); `page.jsx`'s `handleGenerateBadge` URL building, `copyToClipboard`, and `toggleTheme` plus the restore effect; and `route.js`'s success and missing-input branches.

**Proposed Solution**

Add three Jest/RTL test files that mirror `src/app`, in the style of the merged Box Shadow tests. Use the real `NavBar`/`Footer` and the real `badge-maker` (asserting stable SVG fragments, not snapshots), stub only what jsdom lacks (`navigator.clipboard`, `window.alert`), and spy on `console` for the logging the app already does. What I'm deliberately not changing: any app code, including the 500-on-missing-input status and the comma-joined `links` query value (both reported in the PR as observations, one fix per PR); the docs (already done by #692); `npm run lint` (broken on main); dependencies or Jest config.

**Implementation Plan**

1. Add `__tests__/badge-maker/BadgeForm.test.jsx`. Rendering: heading, seven labelled controls, empty defaults, style `flat` with 5 options, submit button. Input: each field updates. Submit: `onSubmit` gets the exact payload, with `links` split, trimmed, and blank lines dropped, plus `logoBase64: ""`, `links: []` and `style: "flat"` when only label and message are filled. `isDarkMode`: `bg-gray-900` vs `bg-gray-100`.
2. Add `__tests__/badge-maker/page.test.jsx`. Initial render with no preview. Submit flow: the preview `<img>` `src` and the Markdown both equal `${window.location.origin}/api/badge?` + `URLSearchParams(data)`. Copy to Clipboard success (writes the Markdown, alerts) and failure (logs, no alert). Theme toggle saved, and a saved theme restored.
3. Add `__tests__/api/badge/route.test.js` (`@jest-environment node`). Valid input gives 200 `image/svg+xml` with the label and message in the SVG. `color`/`style` params reach the SVG. Missing message, missing label, or no params each give 500 JSON `{"error":"Label and message are required"}`.
4. Verify: before/after `jest --listTests | grep -i badge` and restricted coverage; break one behavior per app file and confirm the matching test fails, then restore; run the manifest checks (`format:check`, `npm test`, `test:coverage`).

Deviation from my claim, recorded before building: the claim said all tests would go under `__tests__/badge-maker/`, but contributing.md says to mirror `src/app`, so the route test goes in `__tests__/api/badge/`. I posted that in the 2026-11-15 follow-up. Deviations during the build are logged in Implementation Notes.

**Toolkit**

- Tool: plan-check, run as `claude "plan-check: grade my plan in plan.md and draft comment in plan-comment.md for issue …/400"` from the clone, with the Unit 6 wild scope. Then a manual voice-guard check on `plan-comment.md` (`check_voice.py --draft`, pass). Per the wild scope, the plan comment stays off the thread.
- What it produced: **accept** on the first run. All 7 required checks passed, plus both preferred ones. It confirmed my source claims (the heading text, 5 style options, the alert and error strings, the `theme` key) and pointed out an untested `catch` in `page.jsx` (line 27, "Error fetching badge:") that's probably unreachable, since nothing is fetched on submit. It also noted the route control run is in `plan.md` but not in my posted comment.
- Difficulties: It reported "no `voice-guide.md` in `~/.claude/skills/plan-check/`", but the Unit 6 symlink is there and resolves (`ls -la` shows it pointing to `~/.claude/voice-guard/voice-guide.md`). Its file listing apparently didn't follow the symlink, so its voice pass didn't run. I also had to run Prettier on my local, git-excluded working files, because `npm run format:check` scans the whole clone and the ship-gate runs it there.
- What changed: I covered the missing voice pass with the guard's manual check (pass). I planned to name the `page.jsx` `catch` as uncovered in the PR if it stays uncovered, rather than contort a test to reach it. No plan revision was needed.

---

## Testing Strategy

Since the "fix" is the tests themselves, I needed three kinds of evidence: the reproduction re-run (the gap is closed), the tests passing, and proof that they actually detect breakage.

1. **Reproduction re-run, before vs. after.** `npx jest --listTests | grep -i badge` printed nothing on main `c9cdcc3`. On the branch it lists `__tests__/badge-maker/page.test.jsx`, `__tests__/badge-maker/BadgeForm.test.jsx` and `__tests__/api/badge/route.test.js`. The restricted coverage command went from 0/0/0/0 on all three files to:
   - `BadgeForm.jsx`: 100% statements, branches, functions and lines.
   - `route.js`: 100% statements and lines, 94.44% branches. The uncovered branch is line 10, the unreachable `|| []` after `getAll`.
   - `page.jsx`: 96.87% statements, 96.77% lines. The uncovered line is 27, the `catch` in `handleGenerateBadge`, which nothing in its `try` can trigger.
2. **Targeted run.** `npm test -- __tests__/badge-maker __tests__/api/badge` gave 3 suites and **22 tests**, all passing:
   - route (7): SVG + `image/svg+xml` + `aria-label="build: passing"`; `color`, `style` (`for-the-badge` gives height 28 and upper-case text) and `links` (`href`) reach the SVG; 500 JSON `{"error":"Label and message are required"}` for missing message, missing label, and both.
   - BadgeForm (8): heading and seven fields with defaults; `isDarkMode` true/false; each field updates; the exact `onSubmit` payload with all fields; the defaults payload; links split, trimmed and stripped of blanks.
   - page (7): no badge before submit; preview `src` and Markdown equal `http://localhost/api/badge?label=build&message=passing&labelColor=&color=&logoBase64=&links=&style=flat`; the preview updates on resubmit; copy success (clipboard + alert) and failure (logged, no alert); theme toggle saved as `"true"`; a saved theme restored.
3. **Mutation check.** I broke one behavior per app file, ran its tests, and restored it with `git checkout`. Removing the links `.filter` failed 2 BadgeForm tests. Dropping `window.location.origin` from the URL failed 3 page tests. Returning 200 on missing input failed all 3 missing-input route tests.
4. **Repo checks (ship manifest), fresh shell, on the branch:**
   - `npm run format:check`: clean.
   - `npm test`: 16 suites, 109 tests, all passing (main: 13 and 87).
   - `npm run test:coverage`: 16 suites, 109 tests, all passing.
   - `npm run lint`: fails identically on main and the branch (Next 16 removed `next lint`). It's excluded in the manifest with a date and reason, and disclosed in the PR.
   - `npm run build`: passes.
   - The Husky pre-commit hook passed on all three commits.
   - CI on PR #704: `prettier` ✓ and `test` ✓ (16/109 in both the test and coverage steps). Vercel shows "Authorization required to deploy", the same on merged outside PRs #696, #701 and #702.

Limits: these are jsdom tests, not a real browser. `navigator.clipboard` and `window.alert` are stubs. I didn't load the page in `npm run dev` and click through it.

**Toolkit**

- Tool: Jest/RTL via `npm test`, coverage via `--collectCoverageFrom`, a scripted mutation loop, and the Unit 7 ship-gate (`check_gate.py --branch`), which runs my `ship-manifest.md` checks and then pr-precheck on the branch, `plan.md`, `test_evidence.md` and `pr_draft.md`.
- What it produced: The before/after and mutation evidence above, saved in `test_evidence.md` in the clone. Ship-gate run 1: **PASS** (3/3 manifest checks ok, pr-precheck accept, all 7 required and both preferred checks passing). Its voice note flagged "Since you've asked for one fix per PR" as not said on #400. Run 2, after my fix: **PASS** again.
- Difficulties: `npm test --verbose` printed no per-test names when not in a terminal, so I pulled them from `jest --json`. `npm run format:check` scans the whole clone, so my untracked, excluded working files (`ship-manifest.md` first) made the gate's Prettier check fail until I formatted them. The first fresh-shell command was blocked by Claude Code's safety check because I'd put an `rm -rf coverage` inside a `zsh -c` script. `coverage/` is gitignored, so I just dropped the removal.
- What changed: I confirmed the maintainer's wording on #391 ("each fix in a separate pr please") and changed the PR text to cite #391, then re-ran the gate. I didn't weaken any check or touch the rubric: pr-precheck's asks matched what this repo really wants.

---

## Implementation Notes

**2026-10-19**

Scout scan (0/5) and four direct-grading batches. Chose WebDevTools #400 and posted my claim (comment 6273810106). Bashamega replied "Feel free to work on it" about 4 hours later.

**2026-11-15 (setup and reproduction)**

Re-checked the thread before starting: still open, no assignee, no other claim, no PR for the Badge Maker. Installed the Unit 6 voice guard and wild scopes, and patched the scout's scope check. Forked and cloned. `npm ci`, and 87/87 tests pass on main `c9cdcc3`. Found that contributing.md now documents running tests (from #692) and says tests mirror `src/app`, which conflicts with my claim's `__tests__/badge-maker/` for the route. Found `npm run lint` broken on main. A throwaway probe showed the ESM-only badge-maker 6 imports fine in Jest and the route returns 500 (not 200, as I'd first written in my voice guide) on missing input. repro-check accepted the claim follow-up and repro report. Posted the voice-guarded follow-up (comment 6907202386).

**2026-11-15 (plan)**

Installed the Unit 7 ship-gate. The manifest has `format:check`, `npm test` and `test:coverage` (from CI, contributing.md and the Husky hook), with `lint` excluded as a baseline failure. Both hooks stopped the `--help` hook test. plan-check accepted `plan.md` on the first run. Decision: test current behavior and leave the 500 status and the comma-joined `links` alone as reported observations. Reason: the maintainer asked on #391 for "each fix in a separate pr".

**2026-11-15 (build)**

Branch `feature/400-badge-maker-jest-tests` (contributing.md's `feature/…` form). I probed badge-maker's real SVG output first so the route assertions use real fragments (`fill="#ff0000"`, `height="28"` + `aria-label="BUILD: PASSING"`, `href=`) instead of snapshots. Wrote the route tests (7) → `a224b0f`, BadgeForm tests (8) → `ad5d77b`, page tests (7) → `8121822`. Prettier reflowed one assertion in the BadgeForm test, so I ran `prettier --write` before committing. No deviation from the plan's files or scope. One addition inside the planned page file: a test that the preview updates on resubmit. The mutation check showed each file's tests catch a real break.

**2026-11-15 (PR)**

Ship-gate PASS twice: once on the first draft, then again after I fixed the #391 attribution. Pushed to my fork using gh's jordannLai credential for that one command. Opened PR #704 through Claude Code: the voice guard passed the body, and the ship-gate hook honored the manual pass. CI `prettier` and `test` both passed. Installed the Unit 8 PR monitor (no existing playbook to protect), and `--state` found PR #704. Built the playbook from five merged outside PRs (first review 4.2–15.8h). The ship-gate hook is user-global, so it also stopped one of my own local story-editing commands: the text mentioned the PR command and had apostrophes it couldn't parse. I moved that edit into a script file.

**Toolkit**

- Tool: Claude Code in the clone (with `CLAUDE.local.md`), git with Husky pre-commit, and the scratch probes described above.
- What it produced: Three focused commits (`a224b0f`, `ad5d77b`, `8121822`), 363 lines added across three test files, no app changes, and this dated log.
- Difficulties: badge-maker's ESM-only package looked like a Jest blocker until the probe showed otherwise. jsdom sanitizes empty `type="color"` values to `#000000`, so the payload tests set colors explicitly or rely on React state. The page's `console.log` on every submit and NavBar's localStorage effect both needed handling in the tests.
- What changed: Probing before asserting became my habit for this PR: badge-maker output, `URLSearchParams` array joining, and the multi-link `href`, each checked with real output before it went into a test or the PR text.

---

## Pull Request

**PR Link:** https://github.com/Bashamega/WebDevTools/pull/704

**PR Description:** Title: "test(badge-maker): add Jest tests for the Badge Maker". It's opened from `jordannLai:feature/400-badge-maker-jest-tests` into `main` and uses the repo's PR template. It contains:

- Description: three new test files mirroring `src/app` (`__tests__/badge-maker/BadgeForm.test.jsx`, `__tests__/badge-maker/page.test.jsx`, `__tests__/api/badge/route.test.js`), what each one covers, and that nothing else changed. It notes the path deviation from my claim (route test under `__tests__/api/badge/`, announced on the issue first), and that the docs ask is already covered by contributing.md's "Run the Tests".
- `Fixes #400`. GitHub links #400 as the closing issue.
- Before/after: no badge tests and 0% coverage on main; after, coverage of 100% for `BadgeForm.jsx`, 100% of lines for `route.js` (one unreachable branch) and 96.77% of lines for `page.jsx` (one unreachable `catch`), each explained. Plus the mutation check (2, 3 and 3 failures, then restored).
- Test results: targeted 3 suites / 22 tests, `format:check` clean, `npm test` and `test:coverage` 16 suites / 109 tests; build passes; `npm run lint` disclosed as broken on main.
- "Found while testing, not changed here": the 500 status for missing input and the comma-joined `links`, left out because the maintainer asked on #391 for "each fix in a separate pr", with an offer to open issues for them.
- "AI use" in my words: Claude Code (Anthropic) read the files with me, drafted the three test files, and ran the checks, coverage and mutation runs; I reviewed every test against the component code and checked the outputs before opening.
- The template's Type of change (both N/A, with reasons), all eight checklist items addressed, and Screenshots N/A.

It has no URLs, which keeps it clear of the repo's `mod.yml` auto-moderator (it closes and locks new-contributor PRs with more than 3 links). The `moderate` check passed.

**Maintainer Feedback:**

- 2026-10-19: On the issue, Bashamega replied "Feel free to work on it" to my claim. This was the go-ahead, not a code review.
- 2026-11-16 (04:38 UTC, 2026-11-15 local): On the PR, none yet from the maintainer. The only activity is from bots. CodeRabbit's automated review said "No actionable comments were generated in the recent review", and all four of its pre-merge checks passed (title, linked issue #400, out-of-scope changes, description). Vercel asked a team member to authorize a deploy. There's nothing to address, so I haven't pushed or posted a reply. Per my playbook, I'll nudge only once, and only if Bashamega hasn't reviewed after 4 quiet days (about 2026-11-20).

**Status:** Awaiting review. PR #704 is open, not a draft, and mergeable, with review decision REVIEW_REQUIRED. CI `prettier` and `test` pass. Vercel's "Authorization required to deploy" is the same gate shown on merged outside PRs.

**Toolkit**

- Tool:
  - The Unit 7 ship-gate hook and the voice guard, which both checked `gh pr create` when I opened the PR through Claude Code.
  - gh, to push with my jordannLai credential, set the default repo to Bashamega/WebDevTools, and verify the PR.
  - The Unit 8 PR monitor (`monitor.py --state`, then a full run) with my `playbook.md`.
- What it produced:
  - PR #704 from `jordannLai:feature/400-badge-maker-jest-tests` into `main`, with the exact title, a body identical to `pr_draft.md`, and "Fixes #400" linked as the closing issue.
  - Hook log lines: the voice guard's "pass", and the ship-gate's "PASS (honored the manual pass)", with fingerprints of the head `8121822`, base `c9cdcc3`, title, body, manifest, plan and evidence.
  - CI green for `prettier` and `test`.
  - The monitor's `run-report.md` (FEEDBACK-IN from bot comments, 1 failing check, REVIEW_REQUIRED) and `drafted-move.md` (a wait note, nothing to post).
- Difficulties: The monitor counts bot comments as "feedback in", and it couldn't name the failing check. I had to check by hand that the failing check was Vercel's authorization gate (`gh pr checks` on #696, #701 and #702 shows the same) and not my code. And my macOS keychain still holds a second GitHub account, so I pushed through gh's credential helper for that one command.
- What changed: Nothing posted. I'm following the draft's wait note and my 4-day threshold, and the bot-vs-maintainer distinction is already a rule in my playbook. If Bashamega or CodeRabbit asks for changes, the plan is one batched push, then a `my-reply.md` in my own words posted through the voice guard.

---

## Learnings and Reflections

**Technical Skills Gained**

- Testing a Next.js App Router route handler with no server: the `/** @jest-environment node */` docblock gives the file Web `Request`/`Response`, and I call `GET(new Request(url))` directly, then assert `status`, `headers.get("Content-Type")`, `text()` and `json()`.
- React Testing Library on client components: querying by label text and role (`getByLabelText("Badge Style")`, `getByRole("button", { name: "Toggle dark mode" })`), `fireEvent.change` on controlled inputs, a `jest.fn()` prop to capture the exact `onSubmit` payload, and `waitFor` for promise-based UI (the clipboard `.then` before `alert`).
- Stubbing what jsdom lacks without mocking the app: `Object.defineProperty(navigator, "clipboard", …)`, `jest.spyOn(window, "alert")`, and spying on `console` both to keep output clean and to assert error logging. I also learned jsdom sanitizes an empty `type="color"` value to `#000000`.
- Asserting on a real third-party output (badge-maker's SVG) with stable fragments instead of snapshots, so a dependency bump doesn't break the tests for no reason.
- Mutation checking: breaking one behavior per file to prove a test can fail, and reading uncovered lines to tell missing tests apart from unreachable code (`page.jsx:27`, `route.js:10`).
- How `next/jest`, `NODE_OPTIONS=--experimental-vm-modules` and an ESM-only dependency fit together, and that a Prettier check over the whole repo also scans untracked local files.

**Challenges Overcome**

- **Finding an available issue at all.** My first scan accepted 0/5 because my ranking couldn't see claims or open linked PRs. I graded site picks directly, found #400, and in Unit 6 replaced my over-strict scope check.
- **"Reproducing" a missing test.** There was no bug, so I had to define honest evidence of absence (test listing, grep, 0% coverage) plus a control run showing the feature works, rather than invent a failure. repro-check accepted it on those terms.
- **My claim vs. the repo's rules.** contributing.md's mirroring rule contradicted the path in my claim, and my claim had no report-back time. I posted a short, voice-guarded follow-up instead of quietly doing something different from what I'd said.
- **A wrong fact of my own.** I first wrote that the route returns 200 on missing input. Reading the code and probing it showed 500, so I fixed my voice-guide example and wrote the tests against the real behavior.
- **Tooling friction.** The ship-gate's Prettier check failed on my own excluded working files. The user-global hook stopped a local edit command. A safety check blocked an `rm -rf` inside a shell script. `--verbose` printed no test names outside a terminal. Each fix is in the Toolkit blocks above: format the local files, move the edit into a script file, drop the unnecessary removal, use `jest --json`.
- **Accuracy in public text.** pr-precheck's voice note caught that I'd attributed "each fix in a separate pr" to the wrong thread. I found the real source (#391) and cited it.

**What I'd Do Differently Next Time**

- Read contributing.md, and the merged PRs it's shaped by, before writing a claim, not after. My claim's test path and missing report-back time both came from claiming first and reading the rules second.
- Fix the scout's ranking (drop issues with open linked PRs or recent claims, stop sorting by recent update, split capped queries) before the next live scan, so I don't spend a whole scan on taken issues.
- Put a report-back time in the original claim. The four weeks between my claim and my first update were longer than I'd want a maintainer to wonder about.
- Format and probe early: run the repo's own checks on a clean tree on day one, and check any third-party output with a quick probe before asserting on it. Both saved me from wrong assumptions this time, but only because I happened to check.

---

## Resources Used

- Issue #400 thread, including my claim, Bashamega's "Feel free to work on it", and my follow-up: https://github.com/Bashamega/WebDevTools/issues/400
- WebDevTools `contributing.md`: the Development Workflow and "Run the Tests" sections (setup commands, the `__tests__/` mirroring rule, the `feature/…` branch form): https://github.com/Bashamega/WebDevTools/blob/main/contributing.md
- WebDevTools `.github/PULL_REQUEST_TEMPLATE.md`, `.github/workflows/jest_tests.yml`, `format.yml` and `mod.yml` (the checks CI runs and the link-count auto-moderator), plus `.husky/pre-commit`.
- Merged PR #701 (Box Shadow Generator tests) as a style and evidence model (clipboard stub, mirrored path, mutation check, AI disclosure): https://github.com/Bashamega/WebDevTools/pull/701. Also the maintainer's review pattern on #693, #696 and #701 ("Please check the code rabbit comments").
- Issue #391, where the maintainer asked for "each fix in a separate pr", which set my out-of-scope rule.
- Testing Library docs, "About Queries" (query by role and label): https://testing-library.com/docs/queries/about
- Jest docs: the `@jest-environment` docblock (testEnvironment) and `jest.spyOn`: https://jestjs.io/docs/configuration#testenvironment-string
- Next.js Route Handlers docs (a route exports `GET(request)` and returns a Web `Response`, which is why the test can call it directly): https://nextjs.org/docs/app/building-your-application/routing/route-handlers
- badge-maker's real SVG output (probed locally with `makeBadge`) for the stable fragments the route tests assert.
- My AI301 tools: issue-scout (scan, direct grading, Unit 6 re-grade), repro-check, plan-check, the voice guard, the ship-gate with pr-precheck, and the PR monitor, plus Claude Code (Anthropic) as the assistant that read the code with me, drafted the tests and ran the checks.
