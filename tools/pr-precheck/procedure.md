# Procedure: how this tool grades a PR package

Follow these steps in order. Do not grade any check until the read
order and evidence gathering are complete. Keep notes as short
labelled lines (`PLAN-STEP-1:`, `HUNK-1:`, `CLAIM-1:`, `ASK-1:`, ...)
so each check can cite them.

## Read order

1. **Mode.** Eval mode if given a package bundle; live mode if given a
   `plan.md`, a branch, a draft PR, test evidence, and an issue URL.
   Live mode only: read `scope.md` first; if the issue or PR target is
   not the scoped repo, or the `Repo:` line is still a placeholder,
   stop and say so without grading. Then read `voice-guide.md` (used
   only in the summary).
2. **Repo facts / repo rules** (eval: `## Repo facts`; live:
   `.github/PULL_REQUEST_TEMPLATE.md`, `CONTRIBUTING.md` or
   `docs/CONTRIBUTING.md`, any `AI_POLICY.md`, the scope's house
   rules). Note every `ASK-n:` (each template section, checklist item,
   issue-link format, changelog/whatsnew requirement, title
   convention), `CHECKS:` any named test/lint/check command the repo
   says must pass (or `none stated`), and `POLICY:` the AI rule
   verbatim and where it demands disclosure (PR / comments / own
   words only / none).
3. **Issue and thread.** Note `BEHAVIOR:` the one behavior the issue
   reports, and every `DIRECTION-n:` explicit maintainer direction or
   question (or `DIRECTION: none`).
4. **Plan context**, before the PR, so the PR's own description cannot
   frame how the plan is read. Note every `PLAN-STEP-n:` (location and
   change), `IN-SCOPE:` items, `NOT-IN-SCOPE:` items, `FILES:` the
   files the plan names, `DEVIATION:` any deviation or deferral note,
   `TEST-PLAN:` each failure mode / observable outcome the test plan
   names, and `EXPECTED:` from the repro evidence.
5. **Diff and commits**, before the description, so the diff is read
   against the plan and not against the author's summary of it. For
   each hunk note `HUNK-n: <file> — <what it changes> — <which
   PLAN-STEP it serves, or "none">`. Note `DEBRIS-n:` for any debug
   output, commented-out code, dead code, unrelated TODO, formatting
   or import churn, or identical remove/re-add lines. Note `COMMITS:`
   the commit messages.
6. **Title and description.** Note every `CLAIM-n:` factual claim
   about what the change contains or excludes, which `ASK-n` items the
   description fills, any `DISCLOSES:` AI-use statement (quoted), and
   any deviation or shortfall the description states.
7. **Test evidence** last. Note `AFTER-n:` each fix-specific observed
   after-result and which `TEST-PLAN` item it re-runs, `CONTROL-ONLY:`
   if an item exercises only an unchanged path, `GENERIC:` any bare
   "tests pass"/"works locally" line, and `CHECKS-SHOWN:` the outcome
   of any repo check command.

## Evidence gathering

For each check (locations per `references/evidence-guide.md`):

- **plan-fidelity**: list every `HUNK-n` whose plan step is "none" and
  check it against `DEVIATION` and the description's stated
  deviations; if uncovered, tag it `extra`. Then list every
  `IN-SCOPE`/`PLAN-STEP` item and tag it `delivered`, `deferred-noted`,
  or `missing`.
- **description-fidelity**: for each `CLAIM-n`, write "CLAIM-n says
  ___; the diff ___ → true / false".
- **decisive-test-evidence**: for each `TEST-PLAN` item, record the
  matching `AFTER-n` or `not re-run`; record whether it exercises the
  changed path or only a control.
- **repo-checks-run**: compare `CHECKS` to `CHECKS-SHOWN`.
- **clean-diff**: list every `DEBRIS-n`. A hunk is not debris when it
  is the fix, its test, or an import/line the fix needs.
- **template-and-asks**: for each `ASK-n`, record `met`, `n/a with
  reason`, or `ignored`; for file asks (changelog/whatsnew), check the
  diff's file list.
- **ai-disclosure**: from `POLICY`, decide `requires-PR-disclosure`
  (all-usage or PR rule; note whether tool/extent are named),
  `own-words-only`, or `none`. Record `DISCLOSES` or `absent`.
- **focused-commits**: quote the least informative commit message.
- **thread-engagement**: for each `DIRECTION-n`, `engaged` or
  `ignored` by the description.

## Check execution

1. Execute checks in rubric order: plan-fidelity, description-fidelity,
   decisive-test-evidence, repo-checks-run, clean-diff,
   template-and-asks, ai-disclosure, then the preferred checks.
2. Grade every check, even after a required check fails; the student
   needs the full list.
3. Apply each pass condition literally to the gathered notes. Grade
   the substance, never length, headings, or tone: a terse description
   that meets every ask passes; a polished one over a drifting diff
   fails.
4. Decision rules per check:
   - plan-fidelity: any hunk tagged `extra` or any plan item tagged
     `missing` → `fail`. `deferred-noted` items are fine.
   - description-fidelity: any `CLAIM-n` marked `false` → `fail`.
   - decisive-test-evidence: any `TEST-PLAN` item `not re-run` without
     a deferral note, evidence that is only `GENERIC`, or only
     `CONTROL-ONLY` items → `fail`.
   - repo-checks-run: `CHECKS` names a command and `CHECKS-SHOWN` is
     empty → `fail`; `CHECKS: none stated` → `pass`.
   - clean-diff: any `DEBRIS-n` → `fail`.
   - template-and-asks: any `ASK-n` `ignored` → `fail`.
   - ai-disclosure: `requires-PR-disclosure` and `absent` (or missing
     a named part) → `fail`; `none` → `pass`; `own-words-only` →
     `pass` unless the description says it is AI-written.
5. **Absent evidence**: if the part a check reads is missing from the
   package entirely (no diff, no test-evidence section), grade
   `unclear` and say what was missing. A present section that proves
   nothing is a `fail`, not `unclear`. In eval mode never infer
   evidence from outside the bundle.
6. A check may be graded from the notes alone; re-read the package
   only when a note is ambiguous or a quote is needed for the
   evidence line.
7. Each check's `evidence` line quotes or names the deciding fact (the
   `extra` hunk, the false claim, the missing re-run, the debris line,
   the ignored ask, the missing disclosure).

## Verdict assembly

1. Apply the rubric's verdict rule: `accept` only if every required
   check is `pass`; any required `fail` or `unclear` → `reject`.
   `unclear` counts as `fail`. Preferred checks never change it.
2. The deciding check is the first failing required check in rubric
   order; quote its evidence first in the summary. List any other
   failing required checks after it.
3. In the readable summary, list each check with its grade on one
   line. Live mode: add voice-guide notes, quoting each rule the draft
   title or description breaks, and note any procedure gap you hit.
4. End with the fenced JSON block from SKILL.md, containing every
   check (required and preferred) with its grade and one-line
   evidence, and the verdict. Nothing after the JSON block.
