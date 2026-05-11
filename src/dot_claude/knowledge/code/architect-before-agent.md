---
slug: architect-before-agent
categories: [architecture, decisions]
priority: 1
description: Design module boundaries, data ownership, and concurrency invariants before writing code — implementation cannot invent coherent architecture.
applies_when:
  - starting a non-trivial feature
  - before first edit on greenfield code
  - defining module boundaries or data ownership
  - scoping what to build vs reject
  - writing or revising CLAUDE.md / project context
related: [galls-law, architecture-decision-records, think-before-coding, simplicity-first, agent-expedient-patterns, velocity-masks-decay]
source: https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/
---

# Architect Before Coding

> Code extends the system it is given. It does not invent coherent ones.

## Key Takeaways

- **Design is a separate step.** Coherent architecture — module boundaries, data ownership, message-passing rules — does not emerge from a feature prompt. Skipping design means accepting whatever local optimum the first plausible implementation lands on.
- **Encode invariants up-front.** A project context file (e.g., `CLAUDE.md`) should declare: architectural boundaries, forbidden data shapes (no positional arrays, no untyped maps for structured data), concurrency rules (who may mutate render-visible state), and explicit non-goals.
- **Rails enable, not just restrict.** A clear invariant set speeds routine features because the design space is pre-pruned. No re-litigating the same architectural questions every turn.
- **Revisit rails when scope shifts.** When the product changes (niche tool → generic dashboard), old invariants may no longer fit. Update the rails before extending further; do not improvise.
- **Reject features that fight the invariants.** If a request requires breaking the architecture to fit, the request needs reshaping — or the invariants need an explicit, recorded revision.

## Source

[I'm going back to writing code by hand](https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/)
