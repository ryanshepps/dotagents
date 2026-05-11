---
slug: agent-expedient-patterns
categories: [design, quality]
priority: 2
description: Watch for god objects, positional arrays, unsafe shared mutation, and conditional sprawl — default path when adding code under time pressure.
applies_when:
  - writing new feature code
  - choosing data representations
  - extending shared state or mutable structs
  - adding behavior to an existing function
  - reviewing structural quality of recent edits
related: [simplicity-first, solid-principles, law-of-demeter, dry-principle, architect-before-agent, velocity-masks-decay]
source: https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/
---

# Expedient Patterns to Avoid

> Path of least resistance for new code is usually wrong long-term.

## Common Defaults to Catch

- **God objects.** State accretes into a single struct ("Model", "Context", "State") because adding a field is cheaper than refactoring. 1,000+ line structs are the tell.
- **Type erosion.** Structured data flattened to `[]string`, `map[string]any`, or positional array indices instead of named typed fields. Loses compiler help, invites stringly-typed bugs.
- **Unsafe shared mutation.** Multiple goroutines/threads write the same field with no synchronization or clear ownership. Easy to write, hard to debug.
- **Conditional sprawl.** New behaviors added as `if` branches inside existing functions rather than new types or strategies — cheap to write, exponential to maintain.

## How to Counter

- Reject diffs that grow the central struct; require a new type instead.
- Name the owner-of-mutation invariant before writing concurrent code.
- Prefer typed fields over positional or stringly-typed shapes by default.
- See [[architect-before-agent]] for encoding these as up-front rails.

## Source

[I'm going back to writing code by hand](https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/)
