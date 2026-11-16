# From Zero Test Coverage to a Merged Open-Source Contribution

## Overview

For my open-source contribution, I worked on WebDevTools issue #400, which asked for Jest tests for the project’s Badge Maker feature. The feature itself was already working, but none of its main files had automated test coverage, meaning future changes could break the form, page behavior, or API route without the existing test suite catching the problem. I added Jest and React Testing Library tests for the Badge Maker form, the page workflow, and the `/api/badge` route. I also used two course tools to verify the contribution before opening it upstream: the staff-provided ship-gate, configured with a ship manifest I wrote, and pr-precheck, graded by a rubric I wrote. The result was PR #704, which passed the repository’s formatting and test checks, was approved by the maintainer, and was merged on November 16, 2026.

PR: https://github.com/Bashamega/WebDevTools/pull/704

## What happened

The first challenge was figuring out how to reproduce an issue that was not actually a broken feature. Issue #400 described missing Jest coverage, so there was no crash or incorrect output to trigger. Instead, I treated the absence of tests as the thing I needed to verify. When I built and tested the contribution on 2026-11-15, Jest on the repository’s main branch collected 13 test suites with 87 passing tests, but none of them covered the Badge Maker. Running coverage specifically against the Badge Maker files showed that all three relevant files were at 0% coverage.

```text
$ npx jest --listTests | grep -i badge || echo "NO badge tests found"
NO badge tests found

$ npm test
Test Suites: 13 passed, 13 total
Tests:       87 passed, 87 total

File            | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
  route.js      |       0 |        0 |       0 |       0 | 1-36
  BadgeForm.jsx |       0 |        0 |       0 |       0 | 4-158
  page.jsx      |       0 |        0 |       0 |       0 | 4-96
```

Before I started writing tests, I also checked the feature’s existing behavior so I would not accidentally test an assumption instead of the code that actually existed. One of my first assumptions was wrong. I initially thought the Badge Maker API returned a 200 response when required input was missing. After reading the route and running a small probe, I found that it actually returned a 500 response with a JSON error. Instead of changing the route, which would have expanded the scope beyond issue #400, I changed my plan and wrote tests against the current behavior. That was an important lesson for me because it showed that working in someone else’s repository requires verifying the code before deciding what the “correct” implementation should be.

I had to make another adjustment after reading the repository’s contribution guide more carefully. In my original claim on 2026-10-19, I said all of the Badge Maker tests would live under `__tests__/badge-maker/`. The repository’s `contributing.md` said tests should mirror the `src/app` path, which meant the API route test belonged under `__tests__/api/badge/`. I corrected the plan and posted a follow-up on the issue rather than silently changing what I had told the maintainer.

The final implementation added three test files. The BadgeForm tests covered rendering, default values, user input, the exact submission payload, links parsing, and dark-mode styling. The page tests covered badge preview generation, updating the preview after another submission, clipboard behavior, alert behavior, and theme persistence. The API route tests covered successful SVG generation, parameters such as color and style, and the current error behavior when required input was missing. I used Claude Code to help read the code, draft the tests, and run the checks, and I reviewed each test against the component behavior and verified the outputs myself before opening the PR.

After the new tests were added, the targeted Badge Maker suite contained 22 passing tests, and the full project test suite increased from 87 tests to 109 tests.

```text
$ npm test -- __tests__/badge-maker __tests__/api/badge
Test Suites: 3 passed, 3 total
Tests:       22 passed, 22 total

$ npm test
Test Suites: 16 passed, 16 total
Tests:       109 passed, 109 total

File            | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
  route.js      |     100 |    94.44 |     100 |     100 | 10
  BadgeForm.jsx |     100 |      100 |     100 |     100 |
  page.jsx      |   96.87 |    88.88 |     100 |   96.77 | 27
```

I also wanted evidence that the tests could detect real regressions rather than simply passing against the current code. To check that, I intentionally broke one behavior in each application file and reran the related tests. Removing the links filtering logic caused two BadgeForm tests to fail. Breaking the page URL generation caused three page tests to fail. Changing the missing-input response from 500 to 200 caused all three related route tests to fail. I then restored the original code.

```text
-      status: 500,
+      status: 200,
  ● Badge API missing input › returns a JSON error when the message is missing
  ● Badge API missing input › returns a JSON error when the label is missing
  ● Badge API missing input › returns a JSON error when both are missing
Tests:       3 failed, 4 passed, 7 total
```

This gave me more confidence that the tests were protecting actual behavior instead of just increasing the test count.

Before opening the pull request, I ran the contribution through the ship-gate, a pre-PR hook the course staff provided. It runs the checks I listed in my ship manifest (formatting, the test suite, and the coverage run) and then runs pr-precheck, using the rubric I wrote, against the branch, plan, evidence, and PR description. One of the useful things it caught was not a code problem at all. In my first PR draft, I wrote wording that made it sound like the maintainer had asked for “one fix per PR” on issue #400. When I checked the source, I found that the maintainer had actually said that on issue #391. I corrected the attribution and reran the ship-gate before opening the PR.

```text
First run, pr-precheck note:
"Since you've asked for one fix per PR". The issue #400 thread you supplied has no such request.

Second run, after the fix:
ship-gate: repo check 1/3 `Prettier formatting`: ok (1s)
ship-gate: repo check 2/3 `Jest test suite`: ok (2s)
ship-gate: repo check 3/3 `Jest suite with coverage`: ok (2s)
ship-gate: PASS. feature/400-badge-maker-jest-tests passed your manifest's checks and your pr-precheck.
**Verdict: accept.** Every required check passes, and so do both preferred checks.
```

The final branch passed the repository’s formatting checks, full test suite, and coverage run. The project’s lint command was already broken on the main branch because it still used `next lint`, which was removed in Next.js 16, so I documented that as an unrelated baseline failure instead of trying to fix it in the same PR. The pull request was opened with only the three new test files and no application-code changes.

On PR #704, the formatting and Jest test checks passed. The Vercel check showed “Authorization required to deploy,” an authorization gate that also appears on merged outside-contributor PRs such as #696, #701, and #702, not a failure in the code. The maintainer approved and merged PR #704 on 2026-11-16 without requesting additional code changes, and issue #400 closed as a result. That gave me a complete open-source contribution from issue selection through reproduction, implementation, validation, review, and merge.

## Limitations and what's next

The biggest limitation of my work is that the component tests run in jsdom instead of a real browser. Browser APIs such as `navigator.clipboard` and `window.alert` had to be stubbed, so these tests do not fully reproduce the way a real user interacts with the Badge Maker in Chrome, Safari, or another browser. I also did not run a full end-to-end browser test that fills out the form, generates the badge, verifies the preview, and copies the Markdown through the actual UI.

If I continued this work, my next step would be to add a browser-level end-to-end test for that full Badge Maker workflow. That would complement the Jest tests by checking the integration between the browser, client-side behavior, and API route in a more realistic environment.

I would also improve the issue-scout tool I used earlier in the course. Its first live scan returned five issues that all turned out to already have active claims or pull requests because the ranking logic could not see those signals. A future version should check for recent claim comments and open linked pull requests before ranking candidates. That would make the tool more useful for real open-source work and reduce the amount of manual screening required.