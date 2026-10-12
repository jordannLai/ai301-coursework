# Evidence guide: where evidence lives in a PR package

Each family below maps to one harness category and to the rubric
checks that read it. For each: where to look in an eval bundle, where
to look in live mode, and what good looks like.

## Plan fidelity (harness category: silent-drift)

Rubric checks: plan-fidelity, description-fidelity.

Where it lives:

- Eval: `## Plan context` holds the accepted plan: its steps, the
  files it names, its "Not in scope" list, its test plan, and any
  deviation or deferral note. `## Candidate PR` → `### Diff` shows
  what actually changed (file headers `--- a/` / `+++ b/` and each
  `@@` hunk). `### Title` and `### Description` hold the fidelity
  claims ("implements the plan exactly", "no changes beyond the
  plan", "the docs now document...").
- Live: `plan.md` (approach, scope, test plan, and its `## Deviations`
  section), the branch diff from `git diff main...HEAD` (three dots,
  run in the working copy) plus `git diff main...HEAD --stat`, and the
  draft title/description file (`pr_draft.md`).

What good looks like: every changed file and hunk serves a plan step,
its tests, or plumbing that step needs, or sits under a deviation
note; every planned item is in the diff or explicitly deferred. Silent
drift in the "more" direction is a new option/flag/setting, a rename
or refactor pass, a rewrite of a helper, or a file the plan never
names, with no note. Silent drift in the "less" direction is a
planned item (a docs change, a second failure mode) missing from the
diff while the description says it is there. A deviation stated in
the plan and the description re-ties the mismatch and is honest work.

## Test evidence (harness category: not-tested)

Rubric checks: decisive-test-evidence, repo-checks-run.

Where it lives:

- Eval: `## Candidate PR` → `### Test evidence`, read against the
  plan context's test plan and repro evidence (the Actual/Expected and
  the failing input), and the repo-facts block's named check commands.
- Live: the captured test output (`test_evidence.md`), read against
  `plan.md`'s test plan and the student's week-2 repro (before
  output), plus the repo's stated checks (Makefile targets, the
  contributing guide's test commands).

What good looks like: the plan's repro (the same input that failed)
re-run on the branch, with the after-result shown and matching the
plan's Expected, and every failure mode the test plan names covered.
The before can be the repro's original output. The repo's named
checks appear with an outcome. Not decisive: "tests pass", "works on
my machine", "verified for a day", a suite count with no
fix-specific case, or a re-run of the control path that never broke.

## Diff quality (harness category: unreviewable)

Rubric checks: clean-diff, focused-commits.

Where it lives:

- Eval: `### Diff` (every `+`/`-` line) and `### Commits`.
- Live: `git diff main...HEAD` and `git log main..HEAD --oneline`.

What good looks like: every changed line is the fix, its test, or a
line the fix needs; the fix is visible at a glance. Debris tells:
debug prints (`DEBUG`, `eprintln!`, `print(`, `console.log`),
commented-out code (old attempts, disabled debug lines), dead or
unused functions/variables (`_unused_*`, `allow(dead_code)`), TODOs
added on unrelated lines, re-indentation or reformatting of untouched
logic, import reshuffles the fix does not need, and blocks removed and
re-added identically. Commit messages like "wip"/"fmt + cleanup"
corroborate debris but do not prove it.

## Standards and comms (harness category: standards-wall)

Rubric checks: template-and-asks, ai-disclosure, thread-engagement.

Where it lives:

- Eval: `## Repo facts` → the "pull requests" line (template
  sections, required checklist, issue-link format, changelog/whatsnew
  asks) and the "contribution policy" line (AI policy and where it
  applies); `## Thread highlights` for maintainer direction; the
  description and diff file list for compliance.
- Live: `.github/PULL_REQUEST_TEMPLATE.md`, `CONTRIBUTING.md` /
  `docs/CONTRIBUTING.md`, any `AI_POLICY.md`, the scope's house rules
  ("the template is always used", disclosure section), and the issue
  thread via `gh issue view`.

What good looks like: every required template section and checklist
item has real content (or a stated "not applicable" with reason), the
issue-link line uses the repo's format (`Closes #N`), any file the
asks require (whatsnew, CHANGELOG) is in the diff, and an AI-use
disclosure appears in the description whenever the policy demands it
for PRs, naming the tool and extent if asked. A statement that the
text is in the author's own words complies with an own-words rule.
Boilerplate or a visibly missing required checklist is the wall.
(Whether the description's claims match the diff belongs to plan
fidelity, above.)
