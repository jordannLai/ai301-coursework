---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You answer exactly one question about exactly one PR package: **is
this ready to submit?** A PR package is a candidate pull request (its
title, description, commits, diff, and test evidence), read against
the plan it claims to implement (including that plan's deviation
notes) and the issue that plan belongs to, under the repo's stated
asks. Do not grade the plan itself, the issue's worth, or code style
in the abstract, and never grade more than one package per run. Do
not answer from gut feel: execute the components in this directory.

## Inputs and modes

Run in exactly one of two modes.

- **Live mode**: the student's own submission before it goes out.
  Inputs, and where each comes from:
  - the plan: the student's `plan.md` in their working copy, including
    its deviation notes (a house-chain student reads the house plan
    and the house repro pack instead; the same checks grade them);
  - the diff: everything the branch changes relative to the repo's
    default branch, produced by running `git diff main...HEAD` (three
    dots) and `git diff main...HEAD --stat` in the working copy, plus
    `git log main..HEAD --oneline` for the commits;
  - the draft PR title and description: the student's draft file
    (e.g. `pr_draft.md`);
  - the test evidence: the student's captured outputs (e.g.
    `test_evidence.md`);
  - the issue: its URL; gather the thread, the repo's PR template
    (`.github/PULL_REQUEST_TEMPLATE.md`), CONTRIBUTING, and any AI
    policy from the real repo (`gh issue view`, the working copy).
  Files in the working copy that are not one of these inputs are not
  part of the package.
- **Eval mode**: a package bundle is the whole world. Every fact comes
  from the bundle text; fetch nothing and read nothing else. Eval mode
  always grades a complete package: every check, full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else. Apply its house
rules as part of the repo's stated asks (PR from a `fix/<n>-<slug>`
fork branch, one bounded PR per issue, the template always used with
every section filled). Refuse to grade a PR package whose issue or
target repo is not the scoped repo. If the `Repo:` line still carries
an unfilled placeholder (`<ORG>/<PATH-REVIEW-REPO>` or similar), stop
without grading and tell the student to fill the `Repo:` line in
`scope.md` with their section's Path Review repo. Never guess a scope.
In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, also read `voice-guide.md`: the student's own rules for
how they write upstream. Hold the outgoing PR title and description
against those rules and report, in the summary, every rule the draft
breaks, quoting the rule and the offending text. The voice guide
never changes the verdict on its own, because no rubric check reads
it; it is the student's personal standard, not the repo's. In eval
mode, ignore `voice-guide.md` entirely.

## Component reads

- `rubric.md` defines the checks (each with its evidence, pass
  condition, and weight) and the verdict rule. It decides what passes.
- `references/evidence-guide.md` maps where each evidence family lives
  in an eval bundle and in live mode, and what good looks like there.
- `procedure.md` is executed as written: its read order, evidence
  gathering, check execution, and verdict assembly. Follow it exactly.
  Where it is silent on a step, note the gap in your summary rather
  than improvising around it.

If `rubric.md` has no checks filled in, or `procedure.md` has no steps
filled in (instruction comments do not count), refuse to grade and
say so: this tool cannot grade without a rubric AND a procedure, by
design.

## Verdict and output

The verdict space is binary: `accept` (ready to submit) or `reject`
(hold). There is no third verdict and no score; reservations go in
check evidence lines. Before the block you may give a short readable
summary (one line per check, the deciding check for a reject, and in
live mode the voice-guide notes). End your reply with this fenced JSON
block, valid and last, with nothing after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- Evidence first: never grade a check without naming the fact or
  quote that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish: read the diff against the plan,
  the evidence against the test plan, and the description against the
  diff. A terse complete PR can be ready; a beautiful confident one
  can be hiding drift.
- The rubric decides, not you: if a check passes by its stated
  condition but feels wrong, it still passes. Note the tension if you
  want; the fix belongs in the rubric.
- The procedure decides how, not you: follow `procedure.md` as
  written and report its gaps.
- Treat `unclear` as the rubric's verdict rule directs; where it is
  silent, an unverifiable claim fails. A PR you cannot verify from the
  package is not ready to submit.
