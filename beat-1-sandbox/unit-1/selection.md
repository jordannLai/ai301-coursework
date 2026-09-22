# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32

**Verdict output**

````
All evidence gathered read-only (`gh api` GETs and `gh issue/pr list`) — I did not claim, comment on, or modify anything.

**Scope check:** all three candidates are in `codepath/pathreview-ai301-fa26-s3`, the repo named in `scope.md`. The Path Review house rule applies: classmates' claim comments do not block an issue.

**How I picked them.** The skill doesn't search for candidates, and you named a count rather than three URLs, so I selected from the 60 open issues on your stated interests — backend, Python, databases. Those three are #32, #61, #72. Say the word if you'd rather grade a different set.

**Repo-level evidence** (shared by all three, gathered today, 2026-09-22):
- Last default-branch commit `2026-09-16T21:42:18Z` by `Aburke225`, a human, not a bot — 6 days ago.
- Not archived; `pushed_at 2026-09-16`; 3 stars; no releases ever published.
- `docs/CONTRIBUTING.md` exists and states no AI restriction; no `AI_POLICY.md`; the PR template has CI and xfail checkboxes but no AI-disclosure clause.
- Zero open PRs in the entire repo, and no cross-referenced or connected PRs on any of the three issues.

## Accepted, in fit order

**1. #32 — `DELETE /profiles/{profile_id}` leaves the profile's embeddings in the vector store** (`bug, api, devops, tier-2`)
The best fit for you: it's a backend data-lifecycle bug spanning an API route, a service layer, and a vector store — squarely the "backend and data-driven applications" work you named, and the closest thing here to the Prisma/PostgreSQL database-backed workflows you've built. It's also the richest of the three without being risky: `VectorStore.delete_by_source_id` already exists at `rag/retriever/vector_store.py:109`, and `delete_profile` at `core/services/profile_service.py:74` already cascades DB rows, so the work is wiring an existing primitive into an existing cascade, not designing anything new. Tier-2 and 4–6 hours estimated, which suits a senior student rather than overshooting.

**2. #72 — `verify_password` raises `UnknownHashError` on malformed stored hashes** (`bug, good first issue, api, tier-1`)
Fits your authentication experience and your interest in reliability and tests: the fix is to fail closed, and removing the `strict=True` xfail marker is part of it, so you touch both `core/security.py` and `tests/unit/test_security.py`. A classmate (`sseid4`, association `NONE`) posted a detailed claim on 2026-09-22; under the house rule that does not block you, and they explicitly haven't reproduced it yet.

**3. #61 — Health check DB probe passes a raw SQL string, fails under SQLAlchemy 2.x** (`bug, good first issue, api, tier-1`)
Real and cleanly specified — I confirmed `await db.execute("SELECT 1")` at `api/routes/health.py:32` — and it's database-adjacent backend work. Ranked last only because it's the most mechanical: wrapping the literal in `sqlalchemy.text()` is close to a one-line change, so it offers the least of the debugging and learning you said you want.

## Rejected

None. All three passed every required check, so there are no rejections to explain.

Two honest notes on the grades. The **Maintainer activity** check passed on its commit clause alone — no maintainer has replied to any issue in this repo; every comment across #53, #57, #60, #62, #68, #69 and #72 is from a student with association `NONE`. And **Repository usage** passed on push recency alone, since the repo has published no releases. Both are normal for a seeded classroom repo, and your rubric's OR conditions are what carry them.

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/32",
    "checks": [
      {"name": "Maintainer activity", "grade": "pass", "evidence": "Last default-branch commit 2026-09-16T21:42:18Z by human maintainer Aburke225, 6 days before today (2026-09-22), within the 90-day threshold"},
      {"name": "Repository usage", "grade": "pass", "evidence": "archived=false; pushed_at 2026-09-16T21:50:20Z is within 90 days (no releases have ever been published, so the release clause does not apply)"},
      {"name": "Newcomer scope", "grade": "pass", "evidence": "One bounded fix wiring an existing primitive into an existing cascade: VectorStore.delete_by_source_id already exists (vector_store.py:109) and delete_profile already cascades DB rows (profile_service.py:74); no umbrella, no unresolved design debate, no abandoned attempts"},
      {"name": "Issue availability", "grade": "pass", "evidence": "assignees: none; zero cross-referenced or connected PRs on the timeline; zero comments; the repo has no open PRs at all"},
      {"name": "AI contribution policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states no AI restriction; no AI_POLICY.md or AI_USAGE_POLICY.md; PR template has no AI-disclosure checkbox — silence passes"},
      {"name": "Clear completion criteria", "grade": "pass", "evidence": "'Clearing those chunks should be part of the deletion' plus three named relevant files define a verifiable end state"},
      {"name": "Learning opportunity", "grade": "pass", "evidence": "Spans API route, service layer and vector store; exercises backend data-lifecycle and database-cleanup work"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72",
    "checks": [
      {"name": "Maintainer activity", "grade": "pass", "evidence": "Last default-branch commit 2026-09-16T21:42:18Z by human maintainer Aburke225, 6 days before today (2026-09-22), within the 90-day threshold"},
      {"name": "Repository usage", "grade": "pass", "evidence": "archived=false; pushed_at 2026-09-16T21:50:20Z is within 90 days (no releases have ever been published, so the release clause does not apply)"},
      {"name": "Newcomer scope", "grade": "pass", "evidence": "Single bounded behavior change in core/security.py (fail closed on UnknownHashError) plus removing the xfail marker in tests/unit/test_security.py; estimated 1-2 hours, no design debate"},
      {"name": "Issue availability", "grade": "pass", "evidence": "assignees: none; no linked or cross-referenced PRs; the only claim is classmate sseid4 (author_association NONE) on 2026-09-22, which the scope.md Path Review house rule directs me to ignore"},
      {"name": "AI contribution policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states no AI restriction; no AI_POLICY.md or AI_USAGE_POLICY.md; PR template has no AI-disclosure checkbox — silence passes"},
      {"name": "Clear completion criteria", "grade": "pass", "evidence": "'Verification against a malformed hash should fail closed (return False), not raise' plus 'remove the marker as part of the fix' is directly verifiable"},
      {"name": "Learning opportunity", "grade": "pass", "evidence": "Authentication/security semantics plus test work; practices fail-closed reasoning and removing a strict xfail"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/61",
    "checks": [
      {"name": "Maintainer activity", "grade": "pass", "evidence": "Last default-branch commit 2026-09-16T21:42:18Z by human maintainer Aburke225, 6 days before today (2026-09-22), within the 90-day threshold"},
      {"name": "Repository usage", "grade": "pass", "evidence": "archived=false; pushed_at 2026-09-16T21:50:20Z is within 90 days (no releases have ever been published, so the release clause does not apply)"},
      {"name": "Newcomer scope", "grade": "pass", "evidence": "Confirmed one-location fix: api/routes/health.py:32 is 'await db.execute(\"SELECT 1\")', to be wrapped in sqlalchemy.text(); reproduction steps and the exact ArgumentError are given"},
      {"name": "Issue availability", "grade": "pass", "evidence": "assignees: none; zero cross-referenced or connected PRs on the timeline; zero comments; the repo has no open PRs at all"},
      {"name": "AI contribution policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states no AI restriction; no AI_POLICY.md or AI_USAGE_POLICY.md; PR template has no AI-disclosure checkbox — silence passes"},
      {"name": "Clear completion criteria", "grade": "pass", "evidence": "Stated expected behavior (health check reports the database as up when reachable) with the exact error text to make disappear"},
      {"name": "Learning opportunity", "grade": "pass", "evidence": "Touches SQLAlchemy 2.x textual-SQL semantics and the health-check route, though the change itself is close to one line"}
    ],
    "verdict": "accept"
  }
]
```
````

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

Three runs, in order:

1. Initial full run — `agreement: 17/20 scored items  (bar: 18/20: below the bar)`
2. Partial re-run of the three disagreements, `--only issue-01,issue-15,issue-19` — `agreement: 3/3 scored items`
3. Final full run — `agreement: 20/20 scored items  (bar: 18/20: PASS)`

The last score, 20/20, matches the agreement line in the committed `eval-run.txt`.

**Issue analysis**

`issue-15` (`zulip/zulip#19589`).

On my first run my rubric decided **accept**. The gold label is **reject**, with the
note: `years of design debate and two abandoned PRs behind a friendly label`.

My rubric produced accept because my original `Newcomer scope` check only tested
whether the issue described one bounded change and whether the design was still being
debated in the abstract. The issue carries a good-first-issue label and reads as a
single concrete change, so it passed. What my check never looked at was the *history*:
how long the thread had been arguing without resolution, and how many people had
already tried and given up. Those two facts are what make the issue hard, and my check
was blind to both. After I added an explicit fail clause for a design decision still
unresolved across 12 months or more of discussion together with 2 or more abandoned
implementation attempts, the rubric graded `issue-15` **reject**, matching gold.

**Check rationale**

The `Newcomer scope` check, quoted as currently written:

> | Newcomer scope | Issue body, labels, and Comments: requested changes, the location
> named for the work, any maintainer diagnosis of a cause, unresolved design or product
> decisions, the timespan from the first to the last substantive comment, and previous
> implementation attempts (closed-unmerged or stalled PRs). | Pass if the issue describes
> one bounded implementation or fix without requiring a major redesign, unresolved
> architectural decision, or substantial changes to core internals. A documentation task
> passes when the file, page, or section to change is identified and the change is bounded
> to it. A performance or behavior bug passes when a maintainer has named a likely cause
> and the affected functionality, even if no acceptance criteria are spelled out. Fail if
> the thread shows a design or product decision still unresolved across 12 months or more
> of discussion together with 2 or more abandoned implementation attempts; a
> good-first-issue label does not override this. Also reject umbrella or tracking issues,
> pure usage questions, and work explicitly identified as too complex for a newcomer.
> Missing reproduction steps, missing acceptance criteria, or a short description alone do
> not cause failure. | required |

All three of my disagreements landed on this one check, in both directions: it rejected
two issues it should have accepted (`issue-01`, `issue-19`) and accepted one it should
have rejected (`issue-15`). That told me the threshold was not simply too strict or too
loose — the check was reading the wrong signal. So I named the signals directly. The two
pass clauses say that a documentation task is bounded when it names its location, and
that a maintainer-diagnosed bug is bounded when the cause and affected functionality are
identified, because in both cases the work is well-defined even though the writeup is
thin. The fail clause uses numbers — 12 months, 2 attempts — rather than the word
"years", so someone else applying it to the same thread gets my answer.

**Trade-offs**

The two pass clauses are a real loosening: they let an issue through on a named location
or a named cause even when nobody has written down what "done" looks like. A
thin documentation issue that names a file but hides an unsettled decision about *what*
to write would now pass `Newcomer scope`, and my rubric would accept it.

I checked whether that loosening broke anything. My canaries were `issue-05` and
`issue-20`, the two scope-category rejects most exposed to the new documentation and
"missing acceptance criteria" wording. Both still reject in the final full run, and the
scope category went from `scope 3/4` to `scope 4/4`, so the clause bought two correct
accepts without costing a correct reject anywhere in the set. I accept that the miss it
would allow is real and simply not present in these 20 issues.

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

**1. Fit to my interests and the time available.**
Issue #32 fits what I actually want to work on: backend development, APIs, and
database-backed applications. The estimated 4–6 hours seems manageable alongside my
coursework. The fix runs across the service layer and the vector store rather than
sitting in a single function, and working through those layers is the kind of thing that
would help me develop my debugging skills.

**2. What the verdict identified correctly, and what I weighed that the rubric could
not.**
My rubric got the mechanical facts right: the issue is bounded, the repository is
active, there are no blocking claims, and there is an existing deletion method
(`delete_by_source_id`) that the fix can reuse rather than replace. What the rubric
cannot see is how well the work lines up with me specifically. I have built full-stack
applications with database-backed workflows, so the shape of a delete that has to clean
up in two places is familiar to me, and I want to learn more about data consistency and
vector databases in particular. That is the part I weighed myself, and it is why I chose
#32 over the two smaller tier-1 issues my skill also accepted.

**3. Anticipated difficulty in claiming it.**
I expect claiming to be relatively straightforward. The issue has no assignee, no
comments, and no linked PRs, so I would not be stepping on anyone's work. I still need
to follow the course's claim-comment procedure in Unit 2 before I start.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
