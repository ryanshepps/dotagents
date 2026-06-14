---
name: code-generate-spec
description: Generate or amend a dated implementation plan. Use after brainstorming and engineering review when the user asks to write a spec, generate a plan/spec, distill a reviewed design from existing code, amend a generated plan, or record a bug/invariant for a planned code change. Produces docs/plans/YYYY-MM-DD-<slug>.md for implementation.
---

# Code Spec Generation

Create or amend a dated implementation plan under `docs/plans/`. This workflow
is adapted from Cavekit's compact spec discipline, but now assumes
`brainstorming` and `code-plan-eng-review` have already shaped the design. The
plan must be explicit enough for implementation: scope, architecture, code
quality, tests, performance, useful review diagrams, and failure modes.

Before drafting or amending a plan, read `references/FORMAT.md`. Use it for
short-plan compression, table-cell rules, symbols, section addressing,
monotonic IDs, task status, and bug rows. This skill overrides only Cavekit's
artifact location and title rule: write dated plans under `docs/plans/`, and use
a human-readable plan title instead of root `SPEC.md` / `# SPEC`.

## Rules

- Write new plans to `docs/plans/YYYY-MM-DD-<topic-slug>.md`.
- Use today's local date for `YYYY-MM-DD`.
- Choose a short, descriptive slug from the requested work, for example
  `2026-06-04-code-generate-spec-skill.md`.
- Create `docs/plans/` if it does not exist.
- Amend the existing plan file the user names. If no file is named, search
  `docs/plans/` for the most relevant dated plan and confirm before editing if
  multiple plausible matches exist.
- Keep required sections in the fixed order below. Add optional sections only in
  their stated positions.
- Preserve code, paths, commands, APIs, env vars, versions, error strings, SQL,
  regex, and URLs verbatim.
- Use the short format from `references/FORMAT.md`, but do not over-compress
  details implementation agents need to reason about.
- IDs are monotonic and never reused: `V1`, `T1`, `B1`, etc.
- `§T` uses status `.` todo, `~` in progress, `x` done.
- Escape literal `|` in table cells as `\|`.
- Do not implement code after generating the spec unless the user explicitly
  asks to continue.

## Dispatch

Inspect the user request and repo state:

1. User describes intended work -> **NEW**.
2. User asks to distill from code -> **DISTILL**.
3. User asks to change a named or discoverable generated plan -> **AMEND**.
4. User reports a bug or failed test for a generated plan -> **BACKPROP**.
5. Existing plan context is ambiguous -> ask which `docs/plans/` file to use.

## Format

Use `references/FORMAT.md` as the base format. Extend it with the implementation
sections below so downstream agents can execute the accepted design without
guessing.

```markdown
# <Plan Title>

Use a human-readable title matching the filename topic, for example
`# Code Generate Spec Skill`.

## §G Goal
One sentence describing the outcome.

## §C Constraints
- Non-negotiable boundaries, stack choices, compatibility requirements, and
  user-stated preferences.

## §E Existing Surface
- Existing code, flows, commands, schemas, tests, or docs that partially solve
  the problem. State whether the plan reuses or changes each one.

## §I Interfaces
- External surfaces the world sees: CLIs, APIs, files, env vars, schemas,
  UI routes, public functions, events, jobs, or generated artifacts.

## §V Invariants
V1: Testable behavior that must hold.
V2: Another invariant.

## §A Architecture
Concrete component boundaries, data flow, ownership, and sequencing. Include
why this is the smallest durable design.

## §T Tasks
id|status|task|cites
T1|.|small, verifiable implementation task|V1,I.api
T2|.|test or validation task|V1

## §Q Test Plan
- Unit, integration, regression, fixture, manual, or command-level checks.
- Name what each test proves and which invariant it covers.

## §F Failure Modes
id|surface|failure|coverage
F1|api: POST /x|timeout after write|test: T2; handling: retry-safe error

## §N Not In Scope
- Explicitly deferred work with one-line rationale.

## §B Bugs
id|date|cause|fix
```

Optional section:

- `## §D Diagrams`: place immediately after `§A` only when an ASCII diagram
  would materially improve human review of non-trivial data flow, state
  machines, dependency graphs, processing pipelines, or decision trees. Do not
  add a diagram merely because another agent will implement the spec.

## NEW

Input: user idea.

1. Search the repo for existing code, docs, tests, commands, and schemas related
   to the idea before drafting.
2. Extract the goal into `§G`.
3. Capture stated and implied constraints in `§C`.
4. List reusable existing surfaces in `§E`; call out any likely duplication.
5. Define external interfaces in `§I`.
6. Propose testable invariants in `§V`.
7. Write architecture in `§A`; add optional `§D` only when useful for human
   review.
8. Break work into ordered tasks in `§T`; include test tasks, not only code
   tasks.
9. Fill `§Q`, `§F`, `§N`, and empty `§B`.
10. Write the dated plan, show the user the path, and ask whether to continue
    to implementation.

## DISTILL

Input: existing repo or code surface.

1. Read README/docs, package manifests, app entrypoints, public routes/commands,
   tests, CI, and TODO markers.
2. Infer `§G`, `§C`, `§E`, `§I`, and `§V` from actual code, not guesses.
3. Mark uncertain claims with `?` in the affected section.
4. Use `§T` for missing work, known TODOs, test gaps, or review follow-ups.
5. Keep `§B` empty unless the user provided concrete bugs.

## AMEND

Input: targeted change such as `amend §V.3`, `amend §T`, or natural language.

1. Read the target plan and section.
2. Make the smallest edit that preserves fixed section order and monotonic IDs.
3. Do not silently rewrite unrelated sections.
4. If the amendment changes behavior, update affected `§V`, `§T`, `§Q`, `§F`,
   and `§N` links so the implementation plan stays coherent.
5. Show the resulting diff summary.

## BACKPROP

Input: bug report, failed test, or production incident.

1. Trace the root cause in code or failure output before editing the plan.
2. Append a `§B` row for every bug.
3. Add a new `§V` invariant when it would catch the bug class in the future.
4. Add or update `§T`, `§Q`, and `§F` so the fix has a test and failure-mode
   coverage.
5. Do not fix code unless the user explicitly asks to continue after the spec
   update.

## Implementation Compatibility Checklist

Before finishing, verify the plan gives downstream implementation enough
material:

- Scope challenge: `§G`, `§C`, `§E`, and `§N` make the minimum viable scope
  clear.
- Architecture: `§I`, `§A`, and optional `§D` name boundaries and data flow.
- Code quality: `§E`, `§A`, and `§T` identify reuse, duplication risks, and
  likely files.
- Tests: `§V`, `§T`, `§Q`, and `§F` connect behavior to tests.
- Performance: `§A` or `§F` names hot paths, concurrency, caching, data
  access, or explains why none apply.
