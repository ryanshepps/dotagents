---
slug: resilience-patterns
categories: [quality, scale, architecture]
priority: 2
description: Defensive patterns — circuit breaker, bulkhead, timeout, retry with backoff — that contain failures in distributed systems.
applies_when:
  - calling remote services
  - designing for failure isolation
  - hardening service-to-service calls
  - reviewing retry/timeout logic
related: [fallacies-of-distributed-computing, cap-theorem]
source: https://github.com/simskij/awesome-software-architecture
---

# Resilience Patterns

> Defensive patterns — circuit breaker, bulkhead, timeout, retry with backoff — that contain failures in distributed systems.

## Key Takeaways

- **Circuit breaker** trips after a threshold of failures and short-circuits further calls for a cooldown window. Stops piling load on a sick downstream and gives it room to recover. Half-open state probes recovery before fully closing.
- **Bulkhead** isolates resources (thread pools, connection pools, quotas) per dependency so one slow downstream cannot exhaust shared capacity and starve unrelated traffic.
- **Timeouts** are non-negotiable on every remote call. No timeout = unbounded wait = thread/socket exhaustion. Set timeouts shorter than upstream caller's timeout to leave room for retries.
- **Retry with exponential backoff + jitter** for transient errors only. Naive retries amplify load and cause thundering herds; jitter spreads retry storms.
- Retries without circuit breakers turn brownouts into outages. Combine: breaker decides *whether* to call, retry decides *how many times*.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — patterns popularized by Nygard's *Release It!* and Hystrix/resilience4j libraries.
