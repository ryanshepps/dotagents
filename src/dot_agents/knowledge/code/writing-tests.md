---
slug: writing-tests
categories: [testing]
priority: 1
description: Test writing principles — black-box, no implementation details, test isolation, editing tests is fine.
applies_when:
  - writing any test
  - reviewing tests
  - refactoring a test suite
  - debugging flaky tests
related: [coding-style, testing-pyramid, pesticide-paradox, sans-io, expectation-tests, check-helper]
---

# Writing Tests

## Best Practices

### Implementation Details

NEVER test implementation details (e.g., internal function calls, private method names, internal data structures). Tests should only fail when a behaviour is broken, not when the implementation behind it changes.

### Black Box

Treat the unit/stack that you are testing as a black box. Pass it inputs and expect outputs. For example, if you are writing an integration test to verify a particular email gets sent, you should pass in the appropriate parameters and mock the email API so that you can capture the expected API request being made. This gives the test the most amount of surface area and has a better likelihood of capturing bugs while being cheap to write.

### Isolation

Tests should not overlap. Each behaviour should be covered by exactly one test so that a regression produces exactly one failure.

### Editing Tests

It is ALWAYS ok to rewrite/refactor tests to make the test suite more robust as you add new features.

### Skip the Ceremony

Plain `assert(actual == expected)` with a context-aware failure message beats fluent-assertion DSLs, mock frameworks, and BDD `Given/When/Then` wrappers. The ceremony adds vocabulary without adding signal. Reach for mocks only when the seam is genuinely external (network, clock, randomness); if you're mocking your own logic, the architecture is wrong — extract a pure core instead (see [[sans-io]]).

### Awaitable Background Work

If a test triggers asynchronous work, the production API must expose a way to await its completion. Spawning fire-and-forget tasks during a test makes the whole suite flaky — assertions race against work that may not have happened yet. Structured concurrency (every spawn has an owning scope you can join) is a testability requirement, not just a style preference.
