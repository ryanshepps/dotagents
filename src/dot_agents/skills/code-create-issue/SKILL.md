---
name: code-create-issue
description: Create repository-native implementation issues. Use when the user asks to create, draft, file, open, or split GitHub/GitLab/Linear issues for code work. Investigates existing commits, PRs, and issues to infer the tracker and format, avoids duplicates, writes concise context with implementation freedom, adds 1-4 checkbox acceptance criteria, creates the issue in the detected tracker, and reports the resulting URL.
---

# Code Create Issue

Create issues that a future implementer can pick up without needing the full
conversation, while still leaving them room to choose the implementation.

## Rules

- Investigate before writing. Match the repository's existing issue tracker,
  title style, and body format.
- If no issue tracker is evident, assume GitHub.
- Keep the opening context short and useful. Lead with what needs to change and
  why; add code paths only when they help the implementer find the work.
- Do not prescribe the exact implementation unless the user explicitly asks for
  that. Prefer problem statements, constraints, affected surfaces, and expected
  behavior over step-by-step code instructions.
- Put acceptance criteria at the bottom as 1-4 unchecked checkbox questions.
  Each question must be answerable yes/no.
- Do not include agent attribution, command transcripts, or broad investigation
  notes in the issue body.

## Workflow

### 1. Identify Tracker And Conventions

1. Confirm the repository root and current state:
   - `git rev-parse --show-toplevel`
   - `git status --short`
2. Inspect commit history for ticket references, issue closing keywords, and
   title style:
   - `git log --oneline -n 50`
   - Search for patterns such as `#123`, `GH-123`, `ABC-123`, `Closes`,
     `Fixes`, `Resolves`, or tracker URLs.
3. Inspect recent PRs when hosted tooling is available:
   - GitHub: `gh pr list --state all --limit 30 --json number,title,body,url`
   - GitLab: `glab mr list --all --per-page 30` or the closest available JSON
     form
4. Infer the issue tracker:
   - GitHub if PRs or commits reference GitHub issues, `gh` works for the repo,
     or no other tracker is evident.
   - GitLab if merge requests or commits reference GitLab issues.
   - Linear/Jira/etc. if commit messages, PR bodies, or docs consistently use
     their keys or URLs.
5. Inspect recent issues in the detected tracker:
   - GitHub: `gh issue list --state all --limit 30 --json number,title,body,url,state`
   - GitLab: `glab issue list --all --per-page 30` or the closest available JSON
     form
   - Other trackers: use the repository's documented CLI/API if available.

If hosted access fails, use the evidence available in commits, PR bodies, docs,
and local issue templates. If there is still no evidence, use GitHub with the
simple fallback format below.

### 2. Avoid Duplicates

Before creating a new issue, search existing open issues for overlapping scope:

- GitHub: `gh issue list --state open --search "<keywords>" --json number,title,url`
- GitLab or other trackers: use the equivalent search.

If a likely duplicate exists, report it and ask before creating another issue.
If there are related but distinct issues, reference them only when it helps
clarify scope.

### 3. Draft The Issue

Mirror the dominant existing issue format. If no format exists, use:

```markdown
## Summary

<brief context: what needs to change, why it matters, and where to start if a
specific code/docs surface is relevant>

## Acceptance Criteria

- [ ] Can the implementer answer yes to this concrete outcome question?
- [ ] Can the implementer answer yes to this validation or behavior question?
```

Good issue bodies:

- Start with a concise summary/context section.
- Name relevant files, commands, APIs, UI surfaces, schemas, or docs only when
  they are useful entry points.
- Explain constraints the implementer must preserve.
- Separate separate problems into separate issues unless the user asks for an
  umbrella issue.
- Use acceptance criteria to define success, not to dictate the approach.

Avoid:

- Long code archaeology that belongs in a plan, PR review, or implementation
  notes.
- Pseudocode, exact function shapes, or a task checklist that removes the
  implementer's freedom.
- Acceptance criteria that are vague, multi-part, or impossible to answer yes/no.

### 4. Create The Issue

Create the issue in the detected tracker using its normal CLI/API:

- GitHub: `gh issue create --title "<title>" --body-file <file>`
- GitLab: `glab issue create --title "<title>" --description-file <file>`
- Other trackers: use the repository's observed tool or API.

Use a temporary body file when possible so quoting and markdown are preserved.
If creating multiple issues, create them one at a time unless the tracker and
environment make parallel creation reliable, then verify the final issue numbers
and URLs.

### 5. Verify And Report

After creation, read the created issue back from the tracker and confirm:

- The title and body were saved correctly.
- The body follows the repo's issue format.
- Acceptance criteria are the final section and contain 1-4 checkboxed yes/no
  questions.

Report the issue number, title, and URL. Mention duplicate or access caveats
only when present.
