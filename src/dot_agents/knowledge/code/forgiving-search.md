---
slug: forgiving-search
categories: [ux, design]
priority: 2
description: Search must match how users describe things, not how data is stored — substring matching is not search.
applies_when:
  - building search, autocomplete, or filter UI
  - diagnosing zero-result complaints on known-good data
  - choosing between substring, tokenized, fuzzy, or semantic matching
  - designing product, document, or catalog lookup
related: [postels-law, jakobs-law, mental-model]
---

# Forgiving Search

> Search must match how users describe things, not how data is stored — substring matching is not search.

## Key Takeaways

- `name.includes(query)` fails the moment a user types a superset, reordering, synonym, or typo. "machine hip abduction" never matches a record named "Hip Abduction". Real search is layered, not a single filter.
- Layer in this order: normalize (lowercase, strip diacritics) → tokenize and AND-match across fields → fuzzy per token (Levenshtein/Bitap) → weight by field (name >> aliases > tags > category) → rank by usage signal (favorites, recency, frequency).
- Synonyms and aliases beat algorithms. "RDL" → "Romanian Deadlift", "DB" → "Dumbbell" — no edit-distance metric recovers these. Store an `aliases: string[]` column and seed it.
- Highlight matched tokens in results and surface "did you mean" on near-misses. Users trust search they can audit and recover from.
- Empty state is part of search. Zero results without a recovery path (suggest, create, broaden) is abandonment.
- Reach for Fuse.js or MiniSearch on the client before building bespoke scoring; reach for Postgres trigram, Typesense, or Meilisearch on the server before rolling your own index.

## Source

Synthesized from Postel's Law (liberal input) and search UX practice (Algolia, Meilisearch, Fuse.js conventions).
