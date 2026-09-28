# Rubric: is this reproduction package ready to post?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here. It ships empty on purpose: the judgment is your
work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the claim comment, the repro report's environment
     record, the artifacts read against the issue's description, the
     repo-facts block) or a location from your
     references/evidence-guide.md. "The report" is not a source; "the
     output excerpt read against the error the issue describes" is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (does
     the artifact show the issue's behavior?), never the write-up's
     shape (how many steps it has, how long it is, whether it uses a
     template's headings). Structure-shaped checks are what make
     graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad packages posted. The lecture named the
proof families: the environment is recorded, the steps are complete
and followable, the behavior shown matches the issue (not an adjacent
one), the outcome is stated honestly (an evidenced cannot-reproduce is
a pass, a confident wrong-target is not), and the words respect the
repo's conventions. A rubric that ignores a family will fail eval
packages designed around that family.
-->


# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Environment recorded | Repro report's environment record: repository revision, operating system, runtime and dependency versions, configuration, and relevant services. | Pass if the record identifies the code revision and the environment details that materially affect the behavior, so another contributor can recreate comparable conditions. Do not fail solely because unrelated version information is omitted. | required |
| Followable reproduction steps | Repro report's commands, setup instructions, input data, and references to repository setup documentation. | Pass if another contributor could follow the described sequence from the recorded environment to the reported outcome without inventing missing commands, inputs, configuration, or prerequisites. Existing repository documentation may supply steps when clearly referenced. | required |
| Issue-specific evidence | Repro artifacts, including terminal output, logs, screenshots, test results, or other observations, read against the behavior described in the original GitHub issue. | Pass if the evidence directly tests the behavior described in the issue and supports the reported result. Fail if the artifacts establish only an adjacent problem, an unrelated setup failure, or a different behavior. | required |
| Honest outcome | Repro report's stated outcome, expected versus observed behavior, and supporting artifacts. | Pass if the reported outcome accurately reflects the evidence. A successful reproduction must show the issue's actual behavior. A cannot-reproduce result passes when the report documents a genuine attempt against the correct target and provides evidence of what happened instead. Fail if the report claims reproduction without supporting evidence or presents a different failure as the target bug. | required |
| Repository conventions | Repo-facts block: the contribution and AI-use policy, classified as (A) an explicit disclosure requirement, (B) a conditional AI policy, or (C) no AI policy. Claim comment and repro report: the AI-use disclosure text, claims about completed work, and promises. Voice guide: required communication conventions. | Pass if the comments comply with applicable contribution rules and do not falsely claim completed work or promise an unverified fix. Grade AI use by which policy the repo facts state. (A) Explicit disclosure requirement (the policy requires AI usage to be disclosed, for example "all AI usage in any form must be disclosed, stating the tool used and the extent of the assistance"): pass only if the claim comment or the repro report actually carries a disclosure in the form the policy asks for. Treat a submitted package as AI-assisted work, so an absent disclosure is a fail, not evidence that no AI was used. (B) Conditional AI policy (AI permitted subject to conditions such as testing, human review, or limits on generated code): pass if the comments show the stated conditions were met; an absent disclosure fails only when the policy makes disclosure one of those conditions. (C) No AI policy in the repo facts: silence about AI use is not a failure and does not create a disclosure requirement. | required |
| Issue-specific claim | Claim comment: issue reference, proposed investigation, and intended next steps, read against the original issue description. | Pass if the claim identifies the actual issue and explains the investigation or reproduction work the contributor intends to perform, without asserting that reproduction is complete before evidence exists. | required |
| Evidence clarity | Repro report's explanation of the artifacts, comparison of expected and observed behavior, and references to relevant files or commands. | Pass if the report connects its observations to the conclusion clearly enough for another contributor to understand why the evidence supports the stated outcome. | preferred |

## Verdict rule

Accept the package only if every applicable required check passes.

Reject the package if any applicable required check fails.

Treat an unclear required check as a failure when the available evidence
does not establish that its pass condition is met.

Preferred checks never change the accept or reject verdict. Use them
only to identify potential improvements in packages that otherwise pass.

For claim-only drafts, evaluate the claim and applicable repository
conventions. Mark checks requiring a reproduction report or artifacts
as not yet applicable rather than failing them because the investigation
has not occurred.

For full reproduction packages, evaluate all applicable required checks.

A genuinely attempted, evidenced cannot-reproduce result is eligible
for acceptance. A claim of successful reproduction without evidence
of the original issue's behavior must be rejected.

For each applicable check, report its grade and supporting evidence.
End with the required binary accept or reject verdict in the skill's
specified JSON format.
