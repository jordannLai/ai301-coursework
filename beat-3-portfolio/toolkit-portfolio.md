# Toolkit Portfolio

**Student:** Jordan Lai

## Featured Tools

**My toolkit in one line:** I wrote the judgment (rubrics, a search strategy, a ship manifest,
a review playbook) for a set of checkers that each graded one stage of my first open-source
contribution before anything went public, from picking WebDevTools issue #400 to getting
PR #704 merged.

issue-scout picked the issue, repro-check and plan-check graded my claim and plan, the
ship-gate with pr-precheck checked the PR before I opened it, and the PR monitor told me
when to wait. I'm featuring the ship-gate because it caught something I got wrong in
public text, and the result is a merged PR.

**Ship-gate + pr-precheck (PR #704)**

**What it is, and the judgment hole I wrote:** The ship-gate is a pre-PR hook the course
staff provided. It runs the checks I listed in my ship manifest (`npm run format:check`,
`npm test`, `npm run test:coverage`), then runs pr-precheck, graded by the rubric I wrote,
against the branch, plan, test evidence, and PR description. `npm run lint` went in the
manifest's baseline-failure table, because it already fails on clean main (Next 16 removed
`next lint`).

**How it evolved, dated (2026-11-15):**

- **Friction.** Passing tests can't tell you whether your PR *text* is true. My first PR
  draft made it sound like the maintainer had asked for "one fix per PR" on issue #400.
  Separately, `format:check` scans the whole clone, so my own untracked manifest failed
  the gate's first check until I formatted it.
- **Change.** Ship-gate run 1 technically PASSED, but pr-precheck's voice note flagged the
  line. I checked the source: the maintainer actually said "each fix in a separate pr
  please" on issue **#391**. I changed the PR text to cite #391 and re-ran the gate. I left
  the rubric alone, because it didn't ask for anything this repo doesn't.
- **Result.** Run 2: PASS, verdict accept. PR #704 opened with only three new test files and
  no app changes. CI's prettier and test checks passed. The maintainer approved and merged
  it on 2026-11-16 without asking for code changes, and issue #400 closed. Badge Maker
  tests: 22 targeted; full suite 87 → 109.

**Evidence:**

```text
Ship-gate run 1 (PASS), pr-precheck voice note on my PR draft:
  "Since you've asked for one fix per PR". The issue #400 thread you supplied has no such request.

  (me: checked the threads; the maintainer's words were on #391, so I fixed the PR text)

Ship-gate run 2, after the fix:
  ship-gate: repo check 1/3 `Prettier formatting`: ok (1s)
  ship-gate: repo check 2/3 `Jest test suite`: ok (2s)
  ship-gate: repo check 3/3 `Jest suite with coverage`: ok (2s)
  ship-gate: PASS. feature/400-badge-maker-jest-tests passed your manifest's checks and your pr-precheck.
  **Verdict: accept.** Every required check passes, and so do both preferred checks.

PR #704: approved + merged 2026-11-16 · issue #400 closed · tests 87 → 109
```

**Honest limit:** The catch was a voice note, not a failed check. Run 1 passed with the
wrong attribution still in it. My rubric's `description-fidelity` check only reads claims
about the diff, so a misquoted maintainer can't block the gate. It got fixed because I read
the notes.

**Next:** Make an attribution the supplied thread doesn't contain a required-check failure
in my pr-precheck rubric, so the gate stops instead of just noting it.

## Inventory

- **issue-scout:** searches GitHub for first issues with my search strategy and grades them
  against my rubric. Its first live scan went 0/5; I replaced "Newcomer scope" with
  `scope-feasible`, and #400 re-graded as accept (difficulty: moderate).
- **repro-check:** grades a claim comment plus repro report. It accepted my #400 claim and
  "evidence of absence" repro (no Badge Maker tests, 0% coverage) on the first run, with two
  voice notes I fixed before posting.
- **plan-check:** grades a plan and plan comment. It accepted my plan on the first run (7/7
  required) and pointed out an untested `catch` in `page.jsx`. Its voice pass didn't follow
  my symlinked voice guide, so I ran the voice guard by hand.
- **Voice guard:** a PreToolUse hook that checks my public GitHub text against my voice
  guide before it posts. It passed my claim follow-up and the PR #704 body.
- **PR monitor:** reads the PR's state and drafts my next move from a playbook I built from
  five merged outside PRs. On #704 it drafted a wait note, and the PR merged without a nudge.

## The Arc

**Beat 1**

Date: 2026-10-19

Friction: issue-scout's first live scan accepted 0 of 5, because its ranking couldn't see claim comments or open linked PRs, so every pick was already taken.

Change: I logged the blind spot in `scan-notes.md`, graded hand-picked candidates directly, and on 2026-11-15 replaced the rubric's "Newcomer scope" row with `scope-feasible`.

Result: WebDevTools #400 passed every required check, and the 2026-11-15 re-grade still said accept, difficulty moderate.

**Beat 2**

Date: November 2026 (Unit 6)

Friction: My voice guide's example said the badge API returns 200 on missing input; reading `route.js` and probing it showed 500.

Change: I fixed the example and wrote "test current behavior, don't change the status code" into `CLAUDE.local.md` and my plan.

Result: repro-check accepted my claim and repro report on the first run, and my route tests assert the real 500 response.

**Beat 3**

Date: 2026-11-15

Friction: Ship-gate run 1 passed, but pr-precheck's voice note showed my PR draft credited "one fix per PR" to the #400 thread.

Change: I found the maintainer's actual words ("each fix in a separate pr please") on #391, corrected the PR text, and re-ran the gate.

Result: Run 2 passed, and PR #704 was approved and merged on 2026-11-16; issue #400 closed and the suite went from 87 to 109 tests.

**Beat 4**

Date: 2026-11-15

Friction: The PR monitor labeled #704 "FEEDBACK-IN" from two bot comments and couldn't name the one failing check.

Change: I confirmed by hand that the failing check was Vercel's authorization gate (merged PRs #696, #701, #702 show the same) and followed the monitor's wait note instead of posting.

Result: The next `--state` read showed MERGED and APPROVED; no nudge was ever needed.

## Limitations and What's Next

The gate trusts its notes to be read: advisory voice notes never change the verdict. My
Badge Maker tests run in jsdom, so `navigator.clipboard` and `window.alert` are stubbed and
no real browser is involved. issue-scout's ranking can't see claim comments or open linked
PRs, so its first live scan went 0/5, all on issues that were already taken.

Next: turn misattributed maintainer quotes into a required pr-precheck failure, add a
browser-level end-to-end test for the full Badge Maker workflow, and have issue-scout drop
issues with open linked PRs or recent claims before ranking.

**Tool limits**

ship-gate / pr-precheck: a wrong attribution to the maintainer only produces a voice note, so the gate still passes

issue-scout: ranking can't see claim comments or open linked PRs, so it surfaces issues that are already taken
