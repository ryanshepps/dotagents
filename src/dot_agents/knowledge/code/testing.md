---
description: Test writing principles and patterns.
type: moc
---

# Testing

Principles and architecture-level moves that make test suites fast, reliable, and easy to extend. Reach here when designing how to test something — pyramid shape, sans-io seams, data-driven cases, snapshot/golden values, coverage marks — and when the suite itself feels slow, flaky, or hostile to additions.

## Entries (by priority)

- [[writing-tests]] (p1) — Test writing principles — black-box, no implementation details, test isolation, editing tests is fine.
- [[data-driven-tests]] (p2) — Express test cases as `(input, expected_output)` rows — a table the test runner iterates — so adding coverage means adding data, not code.
- [[expectation-tests]] (p2) — Store expected output inline next to the assertion; provide an `UPDATE_EXPECT=1` mode that rewrites them when behavior changes intentionally.
- [[sans-io]] (p2) — Separate pure logic from I/O — push computation into pure functions, let a thin shell own filesystem, network, time, and randomness.
- [[testing-pyramid]] (p2) — A project should have many fast unit tests, fewer integration tests, and only a small number of UI tests.
- [[check-helper]] (p3) — Wrap the system-under-test in one `check(input, expected)` function so adding a new test is one line — friction is the real reason test coverage stagnates.
- [[coverage-marks]] (p3) — Plant a named mark on a tricky branch in production code; have the test assert the mark fired — proves the test actually exercised the path it claims to.
- [[not-rocket-science-rule]] (p3) — Before merging, run CI on the actual merge commit — not the branch tip — and merge only if it goes green. Keeps main green and forces tests to stay fast.
