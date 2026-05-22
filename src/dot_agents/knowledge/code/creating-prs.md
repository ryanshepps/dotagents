---
slug: creating-prs
categories: [prs]
priority: 1
description: PR creation workflow — ticket criteria, default branch sync, mandatory post-create skills (/review-pr and /production-readiness).
applies_when:
  - creating a pull request
  - preparing to open a PR
  - finalizing a feature branch
related: [surgical-changes, goal-driven-execution]
---

# Creating PRs

Use this rule when creating PRs.

## Ticket criteria

Verify your list of connected mcps to determine whether a user has a ticket tracking mcp enabled. If they do, consider whether a ticket should be created for this PR.

Tickets SHOULD be created if the PR is:
  - Fixing a user facing bug (e.g., your app is deployed already and you're not prototyping)
  - Introducing a brand new feature
  - Refactoring an existing feature

Tickets should NOT be created if the PR is:
  - A small quick fix
  - A style change

If you are unsure whether a ticket should be created you should ask the user.

## Creating the PR

1. Always make sure the default branch is up-to-date before creating a PR.
2. Determine if a ticket should be created (see criteria above). If yes, create it first and link it to the PR.
3. Use `gh pr create --fill` to create the PR when it produces an acceptable short body. If editing the body, keep it concise and project-focused.

## PR title and description

Use conventional commit style for PR titles, such as `feat(memory): add semantic search` or `refactor(billing): simplify gateway setup`. Do not prepend agent/source labels such as `[codex]` unless the user explicitly asks for a prefix.

Keep PR descriptions short and concise. Do not include agent/source attribution, agent workflow notes, or statements that the work was done by Codex or another assistant. Do not include testing instructions, validation steps, command transcripts, or checklist-style evidence unless the user explicitly asks for them. Summarize what changed and why in project terms.

## Post Creation (MANDATORY — DO NOT SKIP)

Immediately after `gh pr create` succeeds, you MUST invoke BOTH skills below. Use the Skill tool twice in the SAME message so they run in parallel. Do NOT run one, wait for it, then run the other. Do NOT skip either one. Do NOT respond to the user until BOTH have been invoked:

1. `/review-pr <PR number>`
2. `/production-readiness <PR number>`

If you only ran one, STOP and run the other immediately before doing anything else.

Once both complete, return the results to the user and offer to make concrete fixes for each issue.
