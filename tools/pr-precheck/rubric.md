# Rubric: is this pull request ready to submit?

Every check reads the PR's artifacts (the diff, the commits, the test
evidence, the title and description) against the plan they claim to
implement, the issue that plan belongs to, and the repo's stated asks.
None of them grades length, headings, or polish: a terse PR that
delivers the plan with observable proof passes, and a long, confident
PR that drifts from the plan, proves nothing, buries the fix, or
ignores the repo's stated asks fails. `references/evidence-guide.md`
says where each piece of evidence lives; `procedure.md` says the order
to gather and grade it.

The honest-outcome rule applies to every check: a shortfall that the
plan (or its deviation notes) and the description both state openly is
part of the scope, not a failure. "Less than everything" is only a
fail when it is silent.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| plan-fidelity | Every file and hunk in the candidate PR's diff, read against the plan context's approach, in-scope list, named files, "not in scope" list, and any deviation note (in the plan or stated in the description). Then every in-scope item of the plan, read back against the diff. | Pass if (a) every hunk in the diff implements a plan step, its regression tests, or plumbing that step directly needs, or is covered by a stated deviation note, AND (b) every in-scope plan item appears in the diff or is explicitly deferred/noted as not delivered in the plan or description. Fail if the diff adds anything the plan does not contain and no note covers: a new config option, flag, setting, or public API; a rewrite or refactor of surrounding code; a renaming pass; a dependency bump; edits to a file the plan never touches; or anything the plan's "not in scope" list excludes. Fail also if a planned item (a docs change, a second failure mode, a test) is missing from the diff with no note. Being adjacent to the fix does not make extra work in scope. | required |
| description-fidelity | Each factual claim in the candidate PR's title and description about what the change does or does not contain ("implements the plan exactly", "no functional changes outside X", "docs now document Y", "adds a test for Z"), read against the diff. | Pass if every such claim is true of the diff. Fail if any claim states more than the diff delivers (a promised docs/test/behavior change absent from the diff) or denies something the diff contains ("no changes beyond the plan" over a diff with extra hunks). A description that says nothing about fidelity has no claim to contradict and passes this check; drift is then graded by plan-fidelity alone. | required |
| decisive-test-evidence | The candidate PR's test-evidence section (live: the captured test output), read against the plan's test plan, the repro evidence's steps and Expected line, and the code path the diff changes. | Pass if the evidence shows the plan's reproduction (or a named regression test asserting the specific fixed behavior) re-run on the fixed code with an observed after-result that matches the plan's expected outcome, AND it covers every distinct failure mode the plan's test plan names (unless one is explicitly deferred). The "before" may come from the plan's repro evidence; the "after" must be shown as output or a stated observed result. Fail if the evidence is only "tests pass", "works on my machine", "verified locally", "colors work now", or a suite count with no fix-specific outcome; if it exercises only a control or an unchanged path (the case that never broke) instead of the reproduced failure; or if a failure mode the test plan names is silently not re-run. | required |
| repo-checks-run | The repo-facts block's stated checks (a named test command such as `yarn test`, `cargo test`, `pre-commit`, "the test suite passes"), read against the test-evidence section. | Pass if the repo states no specific check command or suite requirement, OR the evidence states the outcome of the stated check(s) (pass/fail, a count, or "clean"). Fail if the repo names a check or suite that must pass and the evidence never shows its outcome. A disclosed, explained failure of an unrelated pre-existing check still passes if it is stated plainly. | required |
| clean-diff | Every hunk and line of the candidate PR's diff, plus the commit list. | Pass if every changed line serves the fix, its tests, or plumbing it needs, so a reviewer sees only the change. Fail if the diff contains any debris: a debug print/log (`eprintln!("DEBUG...")`, `print(`, `console.log`) left in, a commented-out line or block of code (old attempt, disabled debug), a dead/unused function or variable (including behind `allow(dead_code)` or `_unused` names), a TODO/FIXME added on unrelated code, re-indentation, re-ordering, or reformatting of lines the fix does not change, an import restructure the fix does not need, or lines removed and re-added identically (pure churn). Commit messages like "wip", "fix", "misc cleanups while debugging" alongside such debris confirm it, but commit messages alone never fail this check. | required |
| template-and-asks | The repo-facts block's PR-template sections and checklist items and the contribution guide's stated PR asks (issue link format like `closes #NNN`, a changelog/whatsnew entry, a conventional-commit title, required sections), read against the title, description, and diff. Live: `.github/PULL_REQUEST_TEMPLATE.md`, CONTRIBUTING, and the scope's house rules, read against the draft. | Pass if every section and checklist item the repo's template or guide states as required is addressed with real content in the PR (filled, or marked not applicable with a reason), and any file the repo's asks require for this kind of change (e.g. a whatsnew/CHANGELOG entry when fixing a bug) is in the diff. If the repo states no template and no PR asks, pass. Fail if a required section or checklist is visibly absent or left as boilerplate, the required issue-link line is missing, or a required changelog/whatsnew entry is absent from the diff. Judge whether the ask is met, never the formatting used to meet it. | required |
| ai-disclosure | The repo-facts block's contribution/AI policy line (live: CONTRIBUTING, AI_POLICY.md, the PR template's disclosure section), read against the PR description. Treat every package as AI-assisted work. | If the policy states no AI rule, pass. If the policy requires disclosing AI usage in PRs (e.g. "all AI usage in any form must be disclosed", "state the tool and extent"), pass only if the description itself discloses that AI was used, how/to what extent, and the tool when the policy asks for it. If the policy only says comments must be in the author's own words, pass unless the description visibly says it was AI-written; a statement that the description is in the author's own words complies. Fail if a required disclosure is absent or omits a part the policy names (tool, extent). | required |
| focused-commits | The commit list. | Pass if commit messages describe the change (one commit per logical step is fine; one squashed commit is fine). Fail if the commits are uninformative ("wip", "fix", "stuff") or mention work outside the PR. | preferred |
| thread-engagement | The thread highlights' explicit maintainer direction or questions, read against the description. | Pass if the thread has no explicit maintainer direction or question, or the description engages it (follows it, answers it, or says why it diverges). Fail if the description ignores an explicit maintainer question or direction. | preferred |

## Verdict rule

- **accept** if every `required` check grades `pass`.
- **reject** if any `required` check grades `fail` or `unclear`.
- `unclear` counts as `fail`: a PR whose readiness cannot be verified
  from the package is not ready to submit. Grade `unclear` only when
  the part of the package a check reads is genuinely absent (no diff,
  no test-evidence section at all), never as a hedge between pass and
  fail.
- `preferred` checks are always graded and reported, but never change
  the verdict.
- A shortfall disclosed in the plan's notes AND restated in the
  description is in scope and never fails plan-fidelity,
  description-fidelity, or decisive-test-evidence on its own.
