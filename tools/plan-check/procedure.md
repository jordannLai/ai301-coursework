# Procedure: how this skill grades a plan package

Follow these steps in order. Do not grade any check until the read
order and evidence gathering are complete. Keep notes as short
labelled lines (`CAUSE-EVIDENCE:`, `CONTROL-1:`, `DIRECTION-1:`, ...)
so each check can cite them.

## Read order

1. **Mode.** Eval mode if given a package bundle; live mode if given a
   `plan.md`, a draft comment, and an issue URL. Live mode only: read
   `scope.md` first; if the issue is not in the scoped repo, or the
   `Repo:` line is still a placeholder, stop and say so without
   grading. Then read `voice-guide.md` (used only in the summary).
2. **Repo facts / repo rules** (eval: `## Repo facts`; live:
   `docs/CONTRIBUTING.md` or `CONTRIBUTING.md`, any `AI_POLICY.md`,
   the PR template). Note `POLICY:` the AI-use rule verbatim and *where*
   it demands disclosure (all usage / issues+comments / PR only /
   none stated), plus any comment rule.
3. **Issue** (title + body). Note `BEHAVIOR:` the one behavior the
   issue reports, in one line.
4. **Repro evidence**, before the plan, so the plan cannot frame how
   you read the evidence (eval: `## Repro evidence`; live: the
   student's posted repro comment on the issue, or the repro quoted in
   the drafts). Note `ACTUAL:`, `EXPECTED:`, every `CONTROL-n:` (what
   was varied and whether it failed), and `ISOLATES:` the component,
   site, or step the controls/steps pin the failure on.
5. **Thread** (eval: `## Thread highlights`; live: issue comments).
   Note every `DIRECTION-n:` from a maintainer (owner/member/
   collaborator/contributor-maintainer): proposed or rejected
   approaches, located culprit, patch/test request, constraint. Note
   every `PRIOR-ART-n:` open PR or prior attempt. Write `DIRECTION:
   none` / `PRIOR-ART: none` if there are none. The student's own
   claim/repro comments are not maintainer direction.
6. **Candidate plan.** Note `CAUSE:`, `APPROACH:` (each step with its
   location), `IN-SCOPE:` (every in-scope item and every approach
   step), `DEFERRED:`, `TESTS:`, `UNKNOWNS:`.
7. **Candidate plan comment** last. Note `COMMENT-ENGAGES:` which
   DIRECTION/PRIOR-ART items it mentions, and `COMMENT-DISCLOSES:`
   any AI-use statement, quoted.

## Evidence gathering

For each check, the gathering move (all locations per
`references/evidence-guide.md`):

- **grounded-cause**: for each `CONTROL-n`, write one line: "If CAUSE
  were true, CONTROL-n would have ___; it actually ___ → consistent /
  contradicts." Also check whether a step shows the bad state already
  present before the blamed component runs. Compare `CAUSE` to
  `ISOLATES`.
- **fix-targets-cause**: write whether `APPROACH` changes the code at
  `CAUSE`/`ISOLATES`, or only documents, works around, or masks it.
  Check that `EXPECTED` would hold with no user-side workaround.
- **bounded-scope**: list each `IN-SCOPE` item and tag it `needed`
  (the fix, its tests, plumbing the fix requires) or `extra` (refactor,
  rewrite, migration, upgrade, new option/UI, other bugs, harness/CI
  changes, cleanup). Items under `DEFERRED` are not tagged.
- **executable-approach**: for each `APPROACH` step, record the named
  location and the chosen change, or `none`/`undecided`.
- **decisive-test-plan**: from `TESTS`, quote the most specific
  observable outcome; record whether it would differ between broken
  and fixed code and whether it maps to `EXPECTED`.
- **thread-direction**: for each `DIRECTION-n`, record `followed`,
  `diverged-with-reason`, or `ignored`, using the plan AND the
  comment; it counts as engaged only if the comment (not only the
  plan) acknowledges it or follows it recognisably.
- **ai-disclosure**: from `POLICY`, decide `covers-comments`
  (all-usage or issues/comments rule) or `does-not-cover` (none, or
  PR-only with no issue-comment ask). If it covers comments, record
  `COMMENT-DISCLOSES` or `absent`.
- **honest-unknowns**: list claims in the plan the repro did not
  test, and whether each is flagged in `UNKNOWNS`.
- **prior-art**: for each `PRIOR-ART-n`, whether the comment mentions
  it.

## Check execution

1. Execute checks in rubric order: grounded-cause, fix-targets-cause,
   bounded-scope, executable-approach, decisive-test-plan,
   thread-direction, ai-disclosure, then the preferred checks.
2. Grade every check, even after a required check fails; the student
   needs the full list.
3. Apply each pass condition literally to the gathered notes. Grade
   the substance, never length, headings, or tone: a missing heading
   is not a fail if the content is present elsewhere in the plan.
4. Decision rules per check:
   - grounded-cause: any control line marked `contradicts`, or the
     bad state appearing before the blamed component → `fail`.
   - fix-targets-cause: workaround/docs/masking only → `fail`.
   - bounded-scope: any in-scope item tagged `extra` → `fail`, even if
     the core fix is right.
   - executable-approach: any core step with location `none` or
     change `undecided` → `fail`; a single flagged, bounded
     uncertainty about the exact line/layer → still `pass`.
   - decisive-test-plan: no specific observable outcome → `fail`.
   - thread-direction: any `DIRECTION-n` marked `ignored` → `fail`;
     `DIRECTION: none` → `pass`.
   - ai-disclosure: `covers-comments` and `absent` → `fail`;
     `does-not-cover` → `pass`. A disclosure that also says the
     comment is in the author's own words is compliant, never a
     violation.
5. **Absent evidence**: if the part a check reads is missing from the
   package (e.g. no repro evidence quoted at all, no test plan
   section and no test items anywhere), grade `unclear` and say what
   was missing. Missing *plan content* the check requires (no cause,
   no files, no tests) is a `fail`, not `unclear`. Do not infer
   evidence from outside the package in eval mode.
6. A check may be graded from the notes alone; re-read the package
   only when a note is ambiguous or a quote is needed for the
   evidence line.
7. Each check's `evidence` line quotes or names the deciding fact
   (the contradicting control, the `extra` item, the ignored
   direction, the missing disclosure, the observable outcome).

## Verdict assembly

1. Apply the rubric's verdict rule: `accept` only if every required
   check is `pass`; any required `fail` or `unclear` → `reject`.
   `unclear` counts as `fail`. Preferred checks never change it.
2. In the readable summary, list each check with its grade on one
   line, then name the deciding check(s) for a reject with their
   evidence quote. Live mode: add voice-guide notes, quoting each rule
   the draft comment breaks, and note any procedure gap you hit.
3. End with the fenced JSON block from SKILL.md, containing every
   check (required and preferred) with its grade and one-line
   evidence, and the verdict. Nothing after the JSON block.
