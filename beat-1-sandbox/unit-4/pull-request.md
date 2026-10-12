# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s3/pull/164

**Branch**

`fix/32-vector-store-cleanup` (on my fork, `jordannLai/pathreview-ai301-fa26-s3`). The PR fixes issue #32.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

All runs used `run_eval.py` with the installed `~/.claude/skills/pr-precheck/` tool
(SKILL.md, rubric, evidence guide, and procedure), graded by Sonnet. In order:

1. **Run 1, full, saved with `--save-run eval-run.txt`: `agreement: 20/20 scored items  (bar: 18/20: PASS)`.**
   Categories: `clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`.
   No package disagreed.

That was the only eval run. The first full run agreed on every scored package and every
category held its floor, so there was nothing to retry: no `--only` runs, no canaries, and
no second full run. I did not run the calibration packages.

Final score: **20/20**, matching the agreement line in the committed `eval-run.txt`.

Outside the harness, I ran the same tool twice in live mode on my own PR package for #32.
Run 1 gave `accept` with three non-blocking notes, which I fixed in the draft. Run 2 gave
`accept` with no voice-guide breaks. Those are not eval runs and are not in `eval-run.txt`.

**Package analysis**

**`pkg-18` (nushell/nushell#18850, category `unreviewable`).** My rubric decided **reject**.
The gold label is **reject** (gold note: "the is_terminal gate is correct and evidenced, but
the diff drags in an import-restructure hunk, two blocks of identical re-printed lines (pure
churn), and a commented-out debug eprintln; arguable because the debris is mechanical churn
around a right fix").

The grader failed two required checks, `plan-fidelity` and `clean-diff`. Its `clean-diff`
evidence line was: "Commented-out debug line '// eprintln!(\"help colors: tty={}\"...)' left
in; import restructure and identical println! churn; commit '9c37d51 fmt pass on command.rs'."

Why my rubric read it that way: everything else about the package passed. The grader's
`decisive-test-evidence` line was "Piped escape count 214 before, 0 after, plus tty still
colored; matches the plan's expected outcomes", and `description-fidelity`,
`repo-checks-run`, `template-and-asks` and `ai-disclosure` all passed. So the fix itself
was not the problem. `clean-diff` fails on "any debris". Its list names exactly the three
things this diff carries: "a commented-out line or block of code (old attempt, disabled
debug)", "an import restructure the fix does not need", and "lines removed and re-added
identically (pure churn)". My procedure makes the grader note one `DEBRIS-n` line per
item, and its decision rule is "any `DEBRIS-n` → `fail`". The commit `fmt pass on
command.rs` only corroborated the debris. My rubric says commit messages alone never fail
the check, and the preferred `focused-commits` check passed it ("still describes its
change").

`plan-fidelity` failed on the same hunks from the other side: "Import block rewrite (nested
use groups flattened) and identical println! remove/re-add are hunks serving no plan step,
with no deviation note." That is the plan-fidelity rule that every hunk must serve a plan
step. The gold note calls this package "arguable", because the debris is mechanical and the
fix is right. My rubric has no proportionality exception, so one commented-out debug line
holds a correct fix. I agree with the gold label: a reviewer would have to sort mechanical
churn from the real change.

**Check rationale**

From `tools/pr-precheck/rubric.md`, exactly as it reads now:

> | clean-diff | Every hunk and line of the candidate PR's diff, plus the commit list. | Pass if every changed line serves the fix, its tests, or plumbing it needs, so a reviewer sees only the change. Fail if the diff contains any debris: a debug print/log (`eprintln!("DEBUG...")`, `print(`, `console.log`) left in, a commented-out line or block of code (old attempt, disabled debug), a dead/unused function or variable (including behind `allow(dead_code)` or `_unused` names), a TODO/FIXME added on unrelated code, re-indentation, re-ordering, or reformatting of lines the fix does not change, an import restructure the fix does not need, or lines removed and re-added identically (pure churn). Commit messages like "wip", "fix", "misc cleanups while debugging" alongside such debris confirm it, but commit messages alone never fail this check. | required |

Why it reads this way: I did not revise this row. It is the wording from the first and only
eval run, so what shaped it is what I rejected in favour of it.

- **A list of debris tells instead of an adjective.** The starter warned that structure-shaped
  or adjective checks make graders disagree with themselves. A pass condition like "the diff
  is clean" or "the diff is focused" leaves each grader to decide what counts. So the fail
  condition names concrete, observable tells: debug prints, commented-out code, dead or
  `_unused` functions, unrelated TODOs, reformatting of untouched lines, unneeded import
  restructures, and identical remove/re-add lines. Each one is something two graders can
  point at in a `+`/`-` line and agree on. The tells come from the category's own
  description in `gold-labels.json` ("debug prints, dead code, formatting churn, or unrelated
  hunks folded in").
- **"serves the fix, its tests, or plumbing it needs" rather than "only the fix".** A
  regression test file or an import the new code requires is a changed line that is not
  the fix. Without that wording, every PR that adds a test, which is most of the clear
  accepts, would look like debris.
- **Commit messages corroborate but never fail.** I considered failing a PR on "wip"/"fix"
  commit messages, since `pkg-12`'s commits are "wip"/"fix"/"fmt + cleanup". I rejected
  that for the required check. A clean diff with a lazy commit message is still
  reviewable, and in this repo commits can be squashed. Commit quality went into the
  separate *preferred* check `focused-commits`, which never changes the verdict.

**Trade-offs**

**Over-rejects:** `clean-diff` has no proportionality rule. One commented-out line
holds an otherwise correct PR as firmly as a diff full of debris. `pkg-18` is the clearest
case. Its gold note calls it "arguable because the debris is mechanical churn around a
right fix", and my rubric rejects it without weighing the fix against the churn. I accept
that, because the fix for an over-reject here is cheap: the author deletes the line and
pushes again. Letting debris through is worse, because it is the reviewer who pays.

**Under-rejects:** debris that does not look like one of the listed tells gets through.
Examples are a debug statement written as a normal `logger.info(...)` call, a helper that
is technically called but unnecessary, or an unrelated change that looks like real code
rather than churn. `clean-diff` asks whether each line looks like debris. It does not ask
whether the line was needed. An unrelated functional hunk is caught by `plan-fidelity`
("every hunk in the diff implements a plan step... or is covered by a stated deviation
note"), not by this check. So a PR whose plan is vague enough to cover an unrelated hunk
could pass both.

**Nothing changed elsewhere, and how I know:** I never revised this check or any other
check. The first full run was 20/20, with every category at full marks
(`unreviewable 3/3`, `clear-accept 7/7`). So no revision existed that could flip another
package, and no `--only` canaries were needed. The `clear-accept` result is the evidence
that the debris list does not over-fire on clean PRs. All seven clear accepts passed
`clean-diff`, including `pkg-13`, whose grader evidence was "the added comments are
explanatory and every line serves the fix or its test". That shows the check tells
explanatory comments apart from commented-out code. One limit: this rests on a single run,
and grader variance on a borderline package like `pkg-18` could still flip it.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
