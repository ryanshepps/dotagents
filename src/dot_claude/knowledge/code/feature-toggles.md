---
slug: feature-toggles
categories: [quality, planning]
priority: 3
description: Runtime flags that decouple deploy from release, enabling trunk-based development, gradual rollout, and kill switches — but each flag is debt.
applies_when:
  - rolling out risky changes
  - decoupling deploy from release
  - running A/B experiments
  - merging incomplete work to trunk
related: [technical-debt, yagni]
source: https://github.com/simskij/awesome-software-architecture
---

# Feature Toggles

> Runtime flags that decouple deploy from release, enabling trunk-based development, gradual rollout, and kill switches — but each flag is debt.

## Key Takeaways

- **Four common kinds**, each with different lifetime and audience:
  - **Release toggles** — hide in-progress work behind a flag so trunk stays shippable. Short-lived (days–weeks). Delete on launch.
  - **Experiment toggles** — A/B tests. Lifetime = experiment duration. Delete with the verdict.
  - **Ops toggles** — kill switches for risky subsystems (e.g. disable recommendation engine under load). Long-lived; few in number.
  - **Permission toggles** — entitlements per user/tenant. Effectively permanent — not really a toggle, treat as config.
- **Every flag is a code path multiplier.** N flags = up to 2^N runtime configurations. Keep counts small; remove aggressively.
- **Test the on and off path.** CI matrix or contract tests — otherwise the disabled branch silently rots.
- Centralize flag evaluation behind one interface. Avoid `if (config.flagX)` scattered across the code; future you will not find them all.
- Pairs with trunk-based development: merge incomplete features behind a release toggle rather than long-lived branches.

## Source

[awesome-software-architecture](https://github.com/simskij/awesome-software-architecture) — Pete Hodgson's martinfowler.com article is the canonical reference.
