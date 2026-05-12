---
slug: hexagonal-architecture
categories: [architecture, design]
priority: 2
description: Isolate domain logic from I/O by defining ports (interfaces the core needs) and adapters (implementations for DB, HTTP, queues).
applies_when:
  - structuring service boundaries
  - making business logic testable
  - swapping infrastructure (DB, transport)
  - separating domain from framework
related: [solid-principles, law-of-leaky-abstractions]
source: https://github.com/simskij/awesome-software-architecture
---

# Hexagonal Architecture (Ports & Adapters)

> Isolate domain logic from I/O by defining ports (interfaces the core needs) and adapters (implementations for DB, HTTP, queues).

## Key Takeaways

- **Core (domain) depends on nothing external.** Frameworks, DBs, and transports live at the edges. Dependencies point inward — adapters depend on ports, never reverse.
- **Ports** are interfaces owned by the core: driving ports (use cases the core exposes) and driven ports (capabilities the core needs, e.g. `UserRepository`). **Adapters** implement them: HTTP controller drives a use case; Postgres adapter satisfies the repository port.
- Pays off when domain logic is non-trivial and likely outlives any single framework or DB choice. Tests run against fakes wired to ports — no Postgres or HTTP server needed.
- Overkill for thin CRUD services. Adds indirection that earns its keep only when domain rules accumulate or infra churns.
- Same idea as Clean Architecture and Onion Architecture — different vocabulary, identical dependency rule.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — Alistair Cockburn coined the pattern; Bob Martin's Clean Architecture is the same shape.
