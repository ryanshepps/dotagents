---
slug: cqrs-event-sourcing
categories: [architecture]
priority: 3
description: CQRS splits read and write models; event sourcing stores state as an append-only log of events. Powerful, expensive — use only when justified.
applies_when:
  - read/write workloads diverge sharply
  - audit/history is a hard requirement
  - evaluating event-driven architectures
  - designing high-throughput write paths
related: [hexagonal-architecture, premature-optimization]
source: https://github.com/simskij/awesome-software-architecture
---

# CQRS & Event Sourcing

> CQRS splits read and write models; event sourcing stores state as an append-only log of events. Powerful, expensive — use only when justified.

## Key Takeaways

- **CQRS** (Command Query Responsibility Segregation): writes go through a command model optimized for invariants; reads come from one or more denormalized projections optimized for query shape. Lets each side scale and evolve independently.
- **Event sourcing**: persist every state change as an immutable event. Current state = fold over event stream. Gives perfect audit, time-travel debugging, and replay to rebuild projections.
- The two combine well but are independent. CQRS without event sourcing = two models, one DB. Event sourcing without CQRS = single read model derived from events.
- **Costs are real**: eventual consistency between write and read sides, schema evolution on events (you can never rewrite history), more moving parts. Most CRUD apps do not need this.
- Justified when: regulatory audit trail required, temporal queries are core, or read/write scaling differ by orders of magnitude. Otherwise prefer a normal model and revisit later.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — Greg Young (CQRS), Martin Fowler (event sourcing).
