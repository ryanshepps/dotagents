---
slug: architecture-decision-records
categories: [decisions, communication]
priority: 2
description: Short markdown files committed to the repo capturing context, decision, and consequences for each significant architectural choice.
applies_when:
  - making architectural choices
  - onboarding to unfamiliar codebase
  - revisiting a past technical decision
  - documenting tradeoffs
related: [first-principles-thinking, think-before-coding]
source: https://github.com/simskij/awesome-software-architecture
---

# Architecture Decision Records (ADRs)

> Short markdown files committed to the repo capturing context, decision, and consequences for each significant architectural choice.

## Key Takeaways

- **Format is minimal**: Title, Status (proposed/accepted/superseded), Context (forces at play), Decision (what we chose), Consequences (what we accept). One page max.
- **Live next to code**, numbered (`0007-use-postgres-not-mongo.md`). Immutable once accepted — supersede with a new ADR rather than editing. Future readers see the timeline.
- Captures **why**, not what. Code shows the *what*; git history shows *when*; ADR explains the *why* and the alternatives rejected. Prevents revisiting settled debates and helps new joiners understand non-obvious choices.
- Write at the moment of decision, not after. Retroactive ADRs lose the genuine forces and read like rationalization.
- Trigger: any choice that is hard to reverse (DB engine, language, auth model, API style) or that future engineers will question.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — Michael Nygard's 2011 post is the canonical reference.
