---
slug: if-then-sentence-structure
categories: [structure]
priority: 3
description: Frame arguments as logical chains — "If A, then B" — to force precision, expose weak inferences, and let the reader check your reasoning step by step.
applies_when:
  - drafting analytical or persuasive prose
  - writing technical explanations
  - revising arguments that feel hand-wavy
  - making a case in a PR description, design doc, or essay
related: [specificity-beats-abstraction, lead-with-the-news, clarity-over-detail]
source: https://chillphysicsenjoyer.substack.com/p/how-to-write-better
---

# If-Then Sentence Structure

> Frame arguments as logical chains — "If A, then B" — to force precision, expose weak inferences, and let the reader check your reasoning step by step.

## Why

Vague arguments hide their joints. The reader can't tell which step is doing the work, which means they can't agree, disagree, or improve. A sentence shaped like a logical statement — *if X, then Y* — exposes every joint. Each clause is checkable.

This is how mathematicians, scientists, and good lawyers write. Each sentence is an inference; each paragraph is a proof. Even when you're writing prose, borrowing this skeleton sharpens the thinking before it sharpens the language.

## Patterns

The core shape:

- *If [premise], then [conclusion].*

Useful variants:

- *Given [condition], [claim] follows.*
- *Because [cause], [effect].*
- *[Claim], because [reason].*
- *Either [A] or [B]. Not [B], so [A].*
- *[A] implies [B]. We have [A]. Therefore [B].*

Each forces you to name the move you're making.

## Examples

**Hand-wavy:**
> The new caching layer will probably help with latency a lot, and it should also reduce database load, which means we can scale further.

**Logical shape:**
> If the cache hit rate exceeds 80%, p95 latency drops below 100ms. The current workload's request distribution gives us 85% hit rate in staging. So this layer should put us under target — and cut DB queries by ~5x at the same load.

The second version is checkable: you can disagree with the threshold, the staging measurement, or the inference. You can't disagree with "probably help a lot" because there's nothing to grab.

## How to apply

Three moves during revision:

1. **Find the conclusion.** Underline it. If you can't, the paragraph has no claim.
2. **Find the premises.** Each one should be a separately checkable fact or assumption.
3. **Name the inference.** "Therefore," "because," "implies," "follows that." If the inference is missing, the argument is missing.

## When NOT to use

- **Narrative or atmospheric prose.** Stories don't argue; they show.
- **Voice-driven essays** where the meander is the form.
- **Casual surfaces** (Slack, group chat) where the formality reads as condescending.

Strong argumentative writing — design docs, technical explanations, persuasive essays — almost always benefits.

## Source

Adapted from "How to Write Better" — chillphysicsenjoyer.substack.com.
