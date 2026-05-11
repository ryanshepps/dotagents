---
slug: velocity-masks-decay
categories: [quality, decisions]
priority: 2
description: Fast feature shipping hides architectural rot — codebase looks healthy until many things break at once.
applies_when:
  - shipping features fast in same area of code
  - central data structure growing across many edits
  - new features fitting in suspiciously easily
  - judging whether to keep extending vs pause and redesign
related: [technical-debt, broken-windows-theory, lehmans-laws, agent-expedient-patterns, architect-before-agent]
source: https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/
---

# Velocity Masks Decay

> Fast shipping feels like winning right up until everything collapses simultaneously.

## Key Takeaways

- Feature throughput is a misleading health signal. Features land in minutes, tests pass, demo works — none of that measures whether the architecture is still coherent.
- Rot compounds silently. Each expedient choice (mega-struct, scattered conditionals, shared mutation) is locally cheap and globally fatal. The bill arrives when a new feature requires touching every layer at once.
- Treat "this is suspiciously easy" as a warning. If new requests fit the existing shape without friction, either the shape is genuinely good or the shape is being bent to absorb whatever lands.
- Periodically audit structure, not just behavior. Read the central data model. Count fields. Look for type names that mean "everything." Tests cannot catch architectural collapse.

## Source

[I'm going back to writing code by hand](https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/)
