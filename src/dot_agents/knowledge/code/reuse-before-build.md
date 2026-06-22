---
slug: reuse-before-build
categories: [design, quality]
priority: 1
description: Before writing custom code, walk a reuse ladder — necessity, stdlib, platform, existing dependency, one-liner — and only then build.
applies_when:
  - about to write a new function, module, or utility
  - tempted to add a dependency or abstraction
  - reviewing a diff that reimplements existing capability
  - choosing between building and reusing
related: [yagni, simplicity-first, dry-principle, kiss-principle, surgical-changes]
source: https://github.com/DietrichGebert/ponytail/blob/main/skills/ponytail/SKILL.md
---

# Reuse Before Build

> Shortest path to done is the right path. Climb the ladder before writing custom code.

Walk the rungs in order. Stop at the first that solves the problem:

1. **Does it need to exist?** Cut the requirement if nothing breaks. (YAGNI)
2. **Does the stdlib solve it?** Standard library before third-party.
3. **Does a native platform feature cover it?** Framework, runtime, OS, DB.
4. **Can an existing dependency handle it?** Reuse what's already vendored before adding new.
5. **Can it be one line?** Smallest working form wins.

Only after exhausting these do you write custom code.

## Guards

- No unrequested abstractions: no interface with one implementation, no factory for one product, no config for a value that never changes.
- Deletion over addition. Boring over clever — clever is what someone decodes at 3am.
- Adding a dependency is not free: it is supply chain, version churn, and audit surface. A rung-2 stdlib answer usually beats a rung-4 dependency.

Test: can you name which rung your code stops at, and why every rung above failed?

## Source

Ponytail skill — [SKILL.md](https://github.com/DietrichGebert/ponytail/blob/main/skills/ponytail/SKILL.md)
