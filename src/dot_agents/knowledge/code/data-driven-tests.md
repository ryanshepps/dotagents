---
slug: data-driven-tests
categories: [testing]
priority: 2
description: Express test cases as `(input, expected_output)` rows — a table the test runner iterates — so adding coverage means adding data, not code.
applies_when:
  - writing many similar tests
  - covering parsers, formatters, validators, state machines
  - sharing a test corpus across languages or implementations
  - test code is mostly copy-paste with one varying value
related: [check-helper, expectation-tests, sans-io, writing-tests]
source: https://matklad.github.io/2021/05/31/how-to-test.html
---

# Data-Driven Tests

> Cases are data. Adding a test = adding a row.

## Key Takeaways

- A data-driven test is a single test function that loops over a table of cases. Each case is `(input, expected)` — sometimes plus a label.
- Cuts copy-paste, makes the surface area visible at a glance, and lets reviewers see coverage gaps as missing rows rather than missing functions.
- Push the table out of code when it grows: store cases as files in a directory, one fixture per case. The test discovers them. Lets non-coders contribute cases and lets the same corpus drive ports in other languages.
- Pairs naturally with `sans-io` (pure functions are easy to table-test) and `expectation-tests` (each row's expected value can be an inline snapshot).
- Failure message must include the case label or input. Otherwise a red row tells you nothing.

## Source

Alex Kladov (matklad) — [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html)
