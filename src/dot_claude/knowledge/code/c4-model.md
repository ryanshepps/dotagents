---
slug: c4-model
categories: [architecture, communication]
priority: 3
description: Four-level diagram hierarchy — Context, Container, Component, Code — for communicating software architecture at varying zoom levels.
applies_when:
  - documenting system architecture
  - onboarding new engineers
  - explaining a system to stakeholders
  - reviewing service boundaries
related: [architecture-decision-records, conways-law]
source: https://github.com/simskij/awesome-software-architecture
---

# C4 Model

> Four-level diagram hierarchy — Context, Container, Component, Code — for communicating software architecture at varying zoom levels.

## Key Takeaways

- **Level 1 — System Context**: the system as one box, surrounded by users and external systems. Audience: anyone, including non-technical. Answers "what does this thing do and who talks to it."
- **Level 2 — Container**: deployable/runnable units (web app, API, DB, queue) inside the system. Audience: technical staff. Shows tech choices and protocols between containers.
- **Level 3 — Component**: major logical pieces inside one container (controllers, services, repositories). Audience: developers on that container.
- **Level 4 — Code**: class/UML diagrams. Often skipped — IDE shows this, and it rots fast.
- Pairs naturally with ADRs: C4 shows *what* the architecture is; ADRs explain *why* it took that shape. Most teams stop at Levels 1–2 and let code be self-documenting below.
- Notation-agnostic. PlantUML, Structurizr, Mermaid, or whiteboard all work.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — Simon Brown's [c4model.com](https://c4model.com/).
