---
slug: no-half-built-features
categories: [quality, ux, design]
priority: 2
description: Omit unfinished user-facing features unless explicitly asked to expose work in progress.
applies_when:
  - building user-facing product surfaces
  - cutting scope under time pressure
  - wiring UI before backend or workflow support exists
  - deciding whether a placeholder belongs in shipped code
related: [simplicity-first, yagni, principle-of-least-astonishment, peak-end-rule, feature-toggles]
source:
---

# No Half-Built Features

> A feature that advertises itself but cannot complete the user's task is usually worse than no feature.

Do not expose skeleton buttons, empty routes, menu items, settings, or cards that end in "not implemented yet" unless the user explicitly asks for a work-in-progress surface. Unfinished affordances create false expectations, waste clicks, and make the product feel broken even when the rest of the flow works.

Prefer one of three complete states:

- **Leave it out.** If the capability is not wired end to end, remove the entry point from the user-facing surface.
- **Finish the thin slice.** If the feature matters, build the smallest usable path that completes the job.
- **Gate it deliberately.** If the team needs the code merged before launch, hide it behind a feature flag, dev-only route, permission, or internal preview.

This rule is overridable. If the user asks for a prototype, scaffold, disabled placeholder, roadmap preview, or explicit "coming soon" affordance, implement that intent clearly. The default is not "pretend the feature exists"; the default is to protect the user's trust by shipping only coherent, usable surfaces.

Test: if a normal user clicks it, can they accomplish the implied task? If not, either finish it, hide it, or confirm that an unfinished experience is desired.
