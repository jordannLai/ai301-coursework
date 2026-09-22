# Rubric: is this a good first issue?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever checks
you define here. It ships empty on purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where. Name the source
     (repo-facts block, issue body, comment thread, or the locations in
     references/evidence-guide.md). "The repo" is not a source; "the last
     5 default-branch commit dates" is.
   - Pass condition: a condition someone else could apply and get your
     answer. Prefer thresholds with numbers ("a maintainer commented
     within 30 days") over adjectives ("maintainer is responsive").
   - Weight: `required` (a fail here rejects the issue) or `preferred`
     (never changes the verdict; a nice-to-have that helps rank the
     issues you accept).

2. A verdict rule below the table: how the check grades combine into
   accept or reject, including how `unclear` is treated. The verdict
   space is binary. If you write no rule for `unclear`, the skill treats
   it as fail.

Cover what actually kills first contributions. The lecture named four
families: the maintainer is alive, the repo is in use, the scope fits a
newcomer, and nobody else is already on it. A rubric that ignores a family
will fail eval issues designed around that family.
-->



# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer activity | Repo facts: last 5 default-branch commit dates and authors, maintainer first-response sample. Issue Comments: author_association and timestamps. | Pass if a human maintainer committed to the default branch within 90 days of the capture date, or a maintainer responded to an issue within 30 days. A bot-only commit without evidence of human involvement does not pass. | required |
| Repository usage | Repo facts: archived flag, latest release date, last push to any branch, and stars. | Fail if the repository is archived. Otherwise, pass if the latest release occurred within 12 months, the last push occurred within 90 days, or the repository has at least 100 stars together with evidence of ongoing development. | required |
| Newcomer scope | Issue body, labels, and Comments: requested changes, the location named for the work, any maintainer diagnosis of a cause, unresolved design or product decisions, the timespan from the first to the last substantive comment, and previous implementation attempts (closed-unmerged or stalled PRs). | Pass if the issue describes one bounded implementation or fix without requiring a major redesign, unresolved architectural decision, or substantial changes to core internals. A documentation task passes when the file, page, or section to change is identified and the change is bounded to it. A performance or behavior bug passes when a maintainer has named a likely cause and the affected functionality, even if no acceptance criteria are spelled out. Fail if the thread shows a design or product decision still unresolved across 12 months or more of discussion together with 2 or more abandoned implementation attempts; a good-first-issue label does not override this. Also reject umbrella or tracking issues, pure usage questions, and work explicitly identified as too complex for a newcomer. Missing reproduction steps, missing acceptance criteria, or a short description alone do not cause failure. | required |
| Issue availability | Repo facts: this issue's assignees and linked PR states. Issue Comments: claims, PR mentions, timestamps, and maintainer responses. | Pass if there is no active assignee, ongoing implementation claim, or open PR addressing the issue. A closed, unmerged PR or clearly abandoned claim does not automatically fail. When a thread establishes that work has stopped, use that evidence rather than an outdated sidebar assignment. In live Path Review mode, apply the classroom claim exception in scope.md. | required |
| AI contribution policy | Repo facts: contribution policy, including CONTRIBUTING.md, dedicated AI policy files, and applicable PR templates. | Pass if AI-assisted contributions are allowed, are allowed subject to conditions, or no AI restriction is stated. Fail if the repository explicitly prohibits AI-generated contributions. Disclosure, testing, personal understanding, and human-review requirements are conditions, not bans. | required |
| Clear completion criteria | Issue body and Comments: expected behavior, acceptance criteria, and maintainer clarifications. | Pass if the expected result is stated clearly enough to verify whether the requested change has been completed. | preferred |
| Learning opportunity | Issue body: affected functionality, requested work, and applicable labels. | Pass if the issue offers an opportunity to practice coding, debugging, testing, or understanding an unfamiliar codebase. | preferred |

## Verdict rule

Accept an issue if every required check passes.

Reject an issue if any required check fails.

An unclear required check counts as a failure unless the available
evidence establishes that the pass condition is met.

Preferred checks do not affect the accept or reject verdict. Use them
only to help rank issues that have already passed every required check.

Evaluate each issue using the evidence supplied in its snapshot bundle.
Measure recency against the bundle's capture date, not today's date.
In live mode, measure recency against the current date and apply the
Path Review-specific claim exception from scope.md.

For each issue, report the result of every check, the final binary
verdict, and supporting evidence.