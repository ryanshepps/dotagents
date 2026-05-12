---
slug: check-helper
categories: [testing]
priority: 3
description: Wrap the system-under-test in one `check(input, expected)` function so adding a new test is one line — friction is the real reason test coverage stagnates.
applies_when:
  - a test file's setup boilerplate dwarfs the assertion
  - contributors avoid writing tests because the harness is tedious
  - testing a module with a uniform input/output shape
  - building out a test corpus for a parser, evaluator, or formatter
related: [data-driven-tests, expectation-tests, writing-tests]
source: https://matklad.github.io/2021/05/31/how-to-test.html
---

# Check Helper

> One `check` function. New test = one new line. Friction is what kill coverage.

## Key Takeaways

- People skip writing tests when the marginal cost of one more test is high. The fix is structural: invest once in a `check(input, expected)` helper that owns setup, invocation, and assertion. Every subsequent test is a single call.
- Bonus payoff: when the API under test changes, you edit one place — the helper — not 80 tests.
- Pairs with `data-driven-tests` (the loop body is just `check(case.input, case.expected)`) and `expectation-tests` (`check` writes the inline literal on update mode).
- Resist parameter creep. If `check` grows optional flags for every variant, split it into `check_err`, `check_with_config`, etc. Many narrow helpers beat one helper with eight bool args.
- Applies recursively: each architectural layer deserves its own `check`. Don't reuse the top-level integration helper for inner unit tests.

## Source

Alex Kladov (matklad) — [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html)
