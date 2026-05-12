---
slug: coverage-marks
categories: [testing]
priority: 3
description: Plant a named mark on a tricky branch in production code; have the test assert the mark fired — proves the test actually exercised the path it claims to.
applies_when:
  - a test passes whether or not the interesting branch ran
  - protecting a subtle code path from silent removal during refactor
  - documenting why a specific test exists
  - line-coverage tools are too coarse to tell you the right branch executed
related: [expectation-tests, writing-tests]
source: https://matklad.github.io/2021/05/31/how-to-test.html
---

# Coverage Marks

> Tag the path. Assert the tag fires. Tests stop lying about what they cover.

## Key Takeaways

- A coverage mark is a named instrumentation point — e.g. `cov_mark::hit!(empty_input_short_circuit);` — placed on a branch you care about. A matching `cov_mark::check!(empty_input_short_circuit);` in the test asserts that mark was hit during the run.
- Solves a common failure: a test exercises the public API, gets the right answer, but takes a different path than the author thought. The test still passes after a refactor silently removes the branch.
- Doubles as documentation. The mark name + the test it appears in tell future readers what scenario this code path serves.
- Lightweight — no test framework integration needed; a thread-local or atomic counter is enough. Compiles out (or no-ops) in release builds.
- Use sparingly. Only worth it on branches whose existence isn't obvious from the public behavior.

## Source

Alex Kladov (matklad) — [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html)
