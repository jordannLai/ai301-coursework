
# Voice guide: how I talk upstream

## Who I am in threads

I'm a Computer Engineering student with experience in Python, TypeScript, backend development, and database-backed applications. I'm contributing to Path Review to gain experience investigating real issues and working in an existing codebase.

When I comment on an issue, maintainers can expect me to explain what I plan to investigate, provide evidence from my own environment, and communicate what I actually observed without overstating my progress.

## Rules I write by

### Rule: Promise an investigation, not a result

When claiming an issue, I describe what I intend to investigate rather than claiming that I've already reproduced the bug or promising a fix. I don't commit to a completion date before understanding the problem.

- Wrong: "I've confirmed the bug and will have it fixed by tomorrow."
- Right: "I'll investigate the profile deletion workflow, attempt to reproduce the reported behavior, and follow up with my findings."

### Rule: Connect every claim to my own evidence

When reporting a reproduction, I distinguish what I observed from what the original issue describes. I only claim successful reproduction when my own logs, tests, or other artifacts demonstrate the reported behavior.

- Wrong: "The profile deletion bug is definitely reproducible because the issue says embeddings remain."
- Right: "After deleting the test profile, I observed [actual result]. The corresponding output is included below."

### Rule: Report blockers and uncertainty explicitly

If a setup problem prevents me from reaching the reported behavior, I explain the blocker rather than presenting it as evidence of the original bug. If I test the correct behavior but cannot reproduce the issue, I report that honestly.

- Wrong: "The deletion endpoint is broken because my database connection failed."
- Right: "I couldn't reach the deletion endpoint because the database connection failed during setup. I haven't reproduced the reported vector-store behavior yet."

### Rule: Give reproducible details without unnecessary filler

My reproduction comments identify the relevant environment, commands, inputs, and observations so another developer can follow the investigation. I don't replace concrete evidence with generic statements about testing.

- Wrong: "I ran everything locally, and it didn't work."
- Right: "Using [commit], [runtime version], and [relevant configuration], I ran [command] and observed [actual output]."

### Rule: Respect the repository's contribution rules

I follow the project's contribution policy, including any applicable AI-assistance disclosure requirements. I review and verify my work personally and do not imply that another contributor's investigation is my own.

- Wrong: "I copied another student's reproduction, so I can confirm the same result."
- Right: "I followed the repository's setup instructions and tested the issue in my own environment. My observations and supporting output are documented below."

## Things I never post

- A promise to deliver a fix by a particular date before investigating the issue.
- A claim that I've reproduced a bug before actually observing the reported behavior.
- Fabricated commands, logs, test results, screenshots, or environment details.
- Another contributor's reproduction presented as my own work.
- A setup error described as proof of an unrelated application bug.
- A confident conclusion when the evidence is incomplete or inconclusive.
- Generic claims like "everything works" or "the bug is confirmed" without supporting observations.
- A comment that violates the repository's contribution or AI-use policy.
- A promise that I will fix the issue when I have only committed to investigating and reporting it.