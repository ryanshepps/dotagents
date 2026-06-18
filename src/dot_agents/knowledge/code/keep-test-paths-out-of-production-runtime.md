---
slug: keep-test-paths-out-of-production-runtime
categories: [testing, architecture, quality]
priority: 2
description: Keep test and local-only execution paths out of production runtime code; mock external systems at process or protocol boundaries instead.
applies_when:
  - testing integrations with external services
  - adding local development fixtures or fake providers
  - wiring environment-specific runtime behavior
  - reviewing code that branches on test or fixture config
related: [writing-tests, sans-io, hexagonal-architecture, feature-toggles]
---

# Keep Test Paths Out of Production Runtime

> Test inputs should enter through the same boundary production uses.

Do not add fixture, fake-provider, or unit-test-only branches to production
runtime code, even when guarded by environment variables. Env guards still
create runtime modes: on-call engineers must inspect config to know which path
ran, bundles grow, and production can reach dead branches through
misconfiguration.

For external integrations, mock at the edge the app already talks to: HTTP
server, Pub/Sub emulator, SMTP server, filesystem root, clock, or queue adapter.
Production code should still make the same kind of call it makes in real life.
Tests and local development can swap the external endpoint or process, not the
application's internal control flow.

Keep pure domain logic behind ports and test it directly with data. Keep
integration tests black-box: feed inputs at the boundary, observe outputs at the
boundary. If testing requires special fixture branches inside the app runtime,
the boundary is probably misplaced.

Accept small test helpers in test files, factories, or dev-only harnesses.
Reject source changes that make production boot paths choose between "real" and
"fixture" behavior unless the mode is a real product or operations mode with
production ownership, docs, monitoring, and a cleanup plan.
