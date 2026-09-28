
# Evidence guide: where proof lives in a reproduction package

## Environment

**Where it lives:**

- Eval mode: The issue context identifies the affected feature and expected environment. The repro report's environment record identifies the repository revision, operating system, runtime versions, dependencies, configuration, and relevant services. Repo facts and setup documentation provide additional context.
- Live mode: Read the original GitHub issue, the repository's README and setup documentation, and the environment information in the student's draft reproduction report.

**What good looks like:**

The environment identifies the code revision and the software, configuration, and services that materially affect the reported behavior. Another contributor should be able to recreate comparable conditions. Versions should match the issue's stated target, or any relevant differences should be explained.

Do not reject a report solely because it omits unrelated version information.

## Steps

**Where it lives:**

- Eval mode: The repro report's setup instructions, commands, input data, starting state, and action sequence. Read these alongside the original issue context and any referenced repository setup documentation.
- Live mode: The draft reproduction report, commands and files in the student's sandbox, and the repository's installation and testing documentation.

**What good looks like:**

Another contributor can begin from the recorded environment, establish the required starting state, execute the reported actions, and reach the point where the behavior is observed.

Commands, inputs, configuration, and prerequisites must be specific enough that the reader does not need to invent missing steps. Existing repository documentation can provide setup details when the report clearly references it.

The number of steps or use of a particular template does not determine whether the procedure passes.

## Behavior shown

**Where it lives:**

- Eval mode: The original issue's description and expected behavior, compared with the repro report's output excerpts, logs, screenshots, test results, or other artifacts.
- Live mode: The GitHub issue body and relevant clarifications in its thread, compared with the student's actual terminal output, test results, logs, screenshots, or other recorded observations.

**What good looks like:**

The artifacts must test the specific behavior described in the issue. For a successful reproduction, they should demonstrate the reported failure or incorrect state, not merely show that a command ran.

For an honest cannot-reproduce result, the artifacts should show that the correct behavior was tested and what occurred instead.

A setup failure, unrelated exception, or adjacent bug does not establish that the original issue was reproduced.

## Honesty

**Where it lives:**

- Eval mode: Compare the claim comment, the repro report's stated outcome, its expected-versus-observed comparison, and the supporting artifacts.
- Live mode: Compare the student's draft or posted comments with the actual output, logs, and tests from their environment.

**What good looks like:**

The stated conclusion must not exceed what the evidence establishes. A report claiming successful reproduction needs evidence of the issue's actual behavior.

A cannot-reproduce report can pass when it documents a genuine attempt against the correct target, records what was observed, and states relevant limitations.

A blocked investigation should identify the blocker rather than presenting the blocker as proof of the target bug. Do not invent output, test results, or completed investigation steps.

## Comms

**Where it lives:**

- Eval mode: The claim comment and repro report, read against the original issue context, the repo-facts contribution-policy information, and any stated AI-use requirements. Use voice-guide.md for communication conventions.
- Live mode: The original GitHub issue and its comments, the repository's CONTRIBUTING.md, applicable AI policy files and PR templates, and the student's draft claim and reproduction comments.

**What good looks like:**

The claim identifies the specific issue, describes the planned investigation, and promises to report the result without claiming that unperformed work is complete.

The reproduction comment identifies what was tested and accurately communicates the observed outcome and evidence.

Both comments must follow applicable repository contribution rules. Classify the repo facts' AI policy before grading the comments, because the three cases have different pass conditions:

- **Explicit disclosure requirement.** The policy states that AI usage must be disclosed (for example, that all AI usage in any form must be disclosed, naming the tool used and the extent of the assistance). The disclosure must appear in the claim comment or the repro report, in the form the policy asks for. Packages are AI-assisted work, so an absent disclosure is a policy violation, not evidence that no AI was used. A package can be excellent on every proof check and still fail here.
- **Conditional AI policy.** AI assistance is permitted subject to conditions, such as testing the output, human review before submission, or limits on generated code. Check that the stated conditions were met. A missing disclosure is a failure only when disclosure is itself one of those conditions.
- **No AI policy.** The repo facts state no AI rules. Silence about AI use is not a failure and does not create a disclosure requirement.

The comments should communicate the student's actual work rather than relying on generic boilerplate or claiming that another student's evidence is their own.