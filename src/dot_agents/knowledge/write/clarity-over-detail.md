---
slug: clarity-over-detail
categories: [structure]
priority: 2
description: Trade 5% of detail for 20% of clarity. Move supporting nuance to appendices, footnotes, or follow-up sections so the main thread stays readable.
applies_when:
  - revising long-form prose
  - cutting a draft to length
  - structuring technical writing with too many caveats
  - critiquing pieces that drown the point in qualifications
related: [cut-clutter, cut-warmup-openings, lead-with-the-news, if-then-sentence-structure]
source: https://chillphysicsenjoyer.substack.com/p/how-to-write-better
---

# Clarity Over Detail

> Trade 5% of detail for 20% of clarity. Move supporting nuance to appendices, footnotes, or follow-up sections so the main thread stays readable.

## Why

Writers — especially careful ones — pack every paragraph with caveats, edge cases, and parenthetical accuracy. The instinct is good: be honest, be precise. But the result is prose where the spine of the argument is buried under qualifying clauses, and the reader gives up before reaching the point.

A 95%-true claim that 100% of readers understand beats a 100%-true claim that 50% of readers finish. Detail you can't deliver clearly is detail nobody reads.

## The trade

For each caveat, ask:

1. **Does the main claim survive without it?** If yes, move it.
2. **Will most readers care?** If no, move it.
3. **Can a footnote, appendix, or follow-up section hold it?** If yes, move it.

Targets for relocation:

- Edge cases that affect <10% of readers
- Historical context that doesn't change the conclusion
- Methodological caveats that matter to specialists
- Counterarguments worth acknowledging but not engaging
- Definitional precision that interrupts the flow

## Patterns

- **Footnote it.** Inline parentheticals break flow; footnotes preserve the spine and let curious readers dive.
- **Appendix it.** Long technical justifications, full data tables, extended derivations.
- **Defer it.** "I'll come back to X in the next section / post." Promise made, flow preserved.
- **Cut it entirely.** Often the caveat exists because the writer is anxious, not because the reader needs it.

## Examples

**Buried in caveats:**
> The new caching layer (which technically only applies to read-heavy workloads, and even then only when the cache key includes the tenant ID, with the exception of admin endpoints which bypass it) reduces p95 latency by about 40% (though this varies depending on the time of day and the specific endpoint, and we haven't measured it under sustained load yet).

**Clarity first:**
> The new caching layer cuts p95 latency by ~40% on read-heavy workloads.[^1]
>
> [^1]: Applies only to tenant-scoped reads (admin endpoints bypass). Measured at typical load; sustained-load behavior pending.

Same information, but the spine is now legible. The footnote serves readers who care.

## When detail must stay inline

- **Legal, regulatory, or safety claims** where the caveat *is* the claim.
- **Adversarial readers** (court filings, RFPs) who will exploit any imprecision.
- **The detail is the point** — a piece *about* nuance can't relegate the nuance.

Default: clarity wins. Move detail out of the spine.

## Source

Adapted from "How to Write Better" — chillphysicsenjoyer.substack.com.
