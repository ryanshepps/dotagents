---
slug: sans-io
categories: [testing, architecture]
priority: 2
description: Separate pure logic from I/O — push computation into pure functions, let a thin shell own filesystem, network, time, and randomness.
applies_when:
  - designing a module that talks to network or filesystem
  - tests are slow or flaky from I/O
  - refactoring code that's hard to unit test
  - reaching for mocks to test logic
related: [hexagonal-architecture, writing-tests, data-driven-tests]
source: https://matklad.github.io/2021/05/31/how-to-test.html
---

# Sans-I/O

> Pure core, I/O at edges. Test the logic, not the syscalls.

## Key Takeaways

- Test speed and flake are dominated by I/O, not code volume. A million pure-function tests finish in seconds; a hundred network tests take minutes and flake on the CI runner's mood.
- Express the protocol, algorithm, or state machine as pure `data → data` functions. Hand the result to a thin shell that performs the actual syscalls.
- The shell becomes trivial enough that one or two integration tests cover it. The core gets exhaustive, fast unit tests with no fixtures, temp dirs, or network mocks.
- Works across domains: HTTP parsers, database drivers, compilers, game logic, retry/backoff schedulers. If you find yourself reaching for mocks, the seam is in the wrong place — extract a pure core instead.
- Side benefit: pure cores are deterministic. Failures reproduce on the first try.

## Source

Alex Kladov (matklad) — [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html)
