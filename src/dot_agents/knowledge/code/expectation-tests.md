---
slug: expectation-tests
categories: [testing]
priority: 2
description: Store expected output inline next to the assertion; provide an `UPDATE_EXPECT=1` mode that rewrites them when behavior changes intentionally.
applies_when:
  - assertion outputs are large structured strings (formatted code, JSON, ASTs, logs)
  - a deliberate behavior change forces dozens of test edits
  - writing tests for parsers, formatters, codegen, query planners
  - golden-file tests are getting unwieldy
related: [data-driven-tests, coverage-marks, writing-tests]
source: https://matklad.github.io/2021/05/31/how-to-test.html
---

# Expectation Tests

> Snapshot the expected output inline. Let the test rewrite itself when you mean it.

## Key Takeaways

- Also called inline-snapshot or gold-value tests. The expected value lives in the test source — literally next to the call — not in a sibling `.golden` file.
- On mismatch the test fails normally. With an env flag (`UPDATE_EXPECT=1`, `--bless`, `--update-snapshots`), the runner rewrites the expected literal in place. Review the diff like any other code change.
- Authoring cost ≈ zero — write `expect!("")` and run with the flag once. Maintenance cost ≈ zero when intentional changes ripple through 50 tests: rerun, review diff, commit.
- Risk: easy to bless wrong output. Mitigation — always read the diff before committing, never bless on a red CI run, never bless without understanding why the value changed.
- Best for outputs whose shape is hard to assert structurally: pretty-printed trees, error messages, generated code, query plans.

## Source

Alex Kladov (matklad) — [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html)
