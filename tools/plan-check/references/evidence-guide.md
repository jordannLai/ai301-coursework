# Evidence guide: where evidence lives in a plan package

Eval mode: a package bundle has these sections, in this order: header
(`source:`), `## Repo facts`, `## Issue`, `## Thread highlights`,
`## Repro evidence`, `## Candidate plan`, `## Candidate plan comment`.
Live mode: the issue body and thread on GitHub, the student's posted
repro comment on that issue (or the repro quoted in the drafts), the
repo's `CONTRIBUTING`/`docs/CONTRIBUTING.md`, PR/issue templates and
any AI policy file, and the student's `plan.md` and draft comment.

## Diagnosis and grounding

Where it lives:

- The cause: the plan's "Diagnosis" (or first) paragraph; if unlabeled,
  the sentence that says *why* the bug happens.
- What the cause must explain: the repro evidence's **Actual** line
  and artifact (stack trace, output, before/after table).
- What can rule a cause out: the repro evidence's **Control** runs
  (a variant that does NOT fail, or that fails without the blamed
  component), timing matrices, numbered steps that show where the bad
  value first appears (e.g. "at step 4 the value is already wrong,
  before X runs"). Live: the repro comment's control/"to rule out"
  runs, before/after tables, and its "where it comes from" section.

What good looks like: the blamed component is the one the controls
isolate. Test it mechanically: for each control, ask "if the plan's
cause were true, would this control have come out the way it did?"
If any answer is no, the cause is contradicted. If the bad value
already exists at an earlier step than the blamed component runs, the
blamed component cannot be the cause. A diagnosis copied from the
issue or a confident thread comment is still wrong when a control
contradicts it.

The fix must also act at that cause: a docs-only workaround, a
symptom-masking change, or a fix downstream of where the evidence
shows the failure originates does not target the cause.

## Scope

Where it lives: the plan's "Scope" / "In scope" / "Not in scope"
lines, the numbered approach steps, and the list of files. Every
approach step counts as in scope even if the scope line is narrow.

What good looks like: one bounded change: the fix at the isolated
site, its regression tests, and only plumbing the fix itself needs.
Adjacent improvements are named and deferred. A drive-by plan adds
any of: a rewrite/refactor/state-machine redesign of surrounding
code, a dependency upgrade or library migration, a new option,
setting, prop, or UI, a retry framework, a CI matrix or test-harness
migration, fixes for other bugs, or "while I'm in there" cleanups. One
such item fails scope even if the core fix beside it is right and the
plan is polished. A plan that is *narrower* than the full thread
discussion, with the deferral stated and reasoned, is bounded, not
incomplete.

## Executability

Where it lives: the plan's "Approach" / "Files to change" steps.

What good looks like: each step names a location (file, function,
branch, call site) and the concrete change made there; one approach is
chosen. A stated, bounded uncertainty about the exact line or layer,
with how it will be resolved, is fine. Vague looks like: "profile and
optimize", "investigate the input stack", "add recover() somewhere",
"upstream or vendored, whichever is easier", "maybe also check other
X", lists of candidate layers with "not sure", or no files at all.
Every real decision deferred to build time means a stranger cannot
start.

## Test plan

Where it lives: the plan's "Test plan" / "Testing" / "Verification"
section, and test items inside the approach steps (e.g. "add the
issue's cases as regression tests").

What good looks like: an outcome a reviewer could observe flip from
broken to fixed: re-running the repro steps with a stated expected
result (exit 0, the color flips, keys reach `less`, chunk count is 0),
or a named regression test that asserts the reproduced behavior. Map
it onto the repro: the repro's Expected line is what the test should
assert. Not decisive: "run the full test suite", "CI green", "make
sure nothing else breaks", "should feel fast", "manually check it
works" with no stated observable. A generic suite run *in addition*
to a decisive test is fine.

## Honesty

Where it lives: the plan's "Risks" / "Unknowns" / "Open questions" /
"Deviations" text, and hedges inside the approach ("I have not yet
measured...", "may be one layer up..."). Live: a `## Deviations`
section in `plan.md` records mid-build changes.

What good looks like: claims the repro did not test (other platforms,
performance, other code paths) are flagged as unknown with what will
be done about them. False confidence states such things as settled.
A recorded deviation is honest work; a deviation that only exists in
the diff is not.

## Comms

Where it lives:

- Maintainer direction: `## Thread highlights` entries by OWNER,
  MEMBER, COLLABORATOR, or CONTRIBUTOR-maintainers (live: the issue
  thread). Look for: "this seems to be the culprit" + file/line,
  proposed or preferred approaches, approaches called "too expensive"
  or rejected, patched binaries or "please test", constraints.
- Prior art: thread mentions of open PRs (#NNNN), earlier attempts,
  other proposed plans.
- Repo rules: the `## Repo facts` "contribution policy" line (live:
  `CONTRIBUTING`, `AI_POLICY.md`, the PR template). Read *where* the
  policy demands AI disclosure: "all AI usage in any form must be
  disclosed" or "AI-assisted issues and comments" covers the plan
  comment; "state the tool in the pull request" with no ask for issue
  comments does not. Every eval package is AI-assisted work (the code
  or investigation), which is what makes a disclosure rule apply; it
  does not make the comment AI-written. A comment that says it is in
  the author's own words satisfies an own-words rule.
- The words themselves: `## Candidate plan comment` (live: the draft
  comment file).

What good looks like: thread-aware comments name the maintainer's
direction and say "following it" or "diverging because...", mention
open PRs and how this work relates, and carry the disclosure the
policy requires *in the comment* when the policy covers comments.
Boilerplate looks like a comment that could have been posted on any
issue: it restates the plan without reference to anything a
maintainer said, or announces a different approach (e.g. docs-only)
than the one the maintainer already located and patched.
