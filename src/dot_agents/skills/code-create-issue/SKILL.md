---
name: code-create-issue
description: Create repository-native implementation issues. Use when the user asks to create, draft, file, open, or split GitHub/GitLab/Linear issues for code work. Investigates existing commits, PRs, and issues to infer the tracker and format, asks the user to resolve product ambiguity in iterative rounds (looping until no product question remains) before drafting, searches existing open and closed issues and on a likely duplicate asks whether to create a new issue, edit the existing one, or cancel, writes a tight skimmable body in bullet form with implementation freedom, adds 1-4 checkbox acceptance criteria, creates the issue in the detected tracker, and reports the resulting URL.
---

# Code Create Issue

Create issues that a future implementer can pick up without needing the full
conversation, while still leaving them room to choose the implementation.

## Rules

- Investigate before writing. Match the repository's existing issue tracker,
  title style, and body format.
- If no issue tracker is evident, assume GitHub.
- **Resolve ambiguity before drafting — ruthlessly.** Do not guess at unstated
  product decisions, and never punt them into the issue. See "Resolve Ambiguity"
  below.
- **No deferral escape hatches.** If you catch yourself writing "TBD", "to be
  defined later", "the maintainer will decide", "exact X out of scope", or "this
  is the framework, not the final list" — stop. That is an unresolved product
  decision. Go ask it (step 3), then write the concrete answer.
- **Pass the developer test.** A competent implementer should be able to pick up
  the issue and build it without asking a single *product* question. If they'd
  still ask "which ones?", "what exactly happens when?", "what are the values?" —
  the issue is not done.
- **Be specific at product level, never code level.** Name the concrete actions,
  states, values, and behaviors. Do not name functions, types, or write
  pseudocode. PM specificity, not engineer specificity.
- **Hard length caps.** Enforce them:
  - Each bullet: one idea, ≤ ~20 words, ≤ 2 lines. Split or cut if longer.
  - Each section: ≤ 6 bullets. More than that means split the issue.
  - No paragraph longer than one sentence. No multi-sentence Summary intro.
  - Whole body skimmable in ~30 seconds.
- Lead with what needs to change and why; add code paths only when they help the
  implementer find the work.
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

### 2. Check For Existing Issues

Always search before creating — never file a new issue without checking that the
work isn't already tracked. Search both open and closed issues for overlapping
scope:

- GitHub: `gh issue list --state all --search "<keywords>" --json number,title,url,state`
- GitLab or other trackers: use the equivalent search.

Run a few searches with different keywords (feature name, surface, synonyms) so a
differently-worded duplicate still surfaces.

**If a likely duplicate exists, stop and ask the user before doing anything
else.** Show the matching issue(s) (number, title, URL, state) and use
`AskUserQuestion` to let them choose:

- **Create new (continue)** — the existing issue is not actually the same; file
  the new issue anyway.
- **Edit the existing issue** — update that issue instead of creating a
  duplicate. Draft the body per steps 3-4, then update in place
  (`gh issue edit <n> --body-file <file>`, reopening with
  `gh issue reopen <n>` first if it is closed and the work is being revived).
- **Cancel** — stop without creating or editing anything.

Default to recommending "Edit the existing issue" when the overlap is strong.
Do not create a second issue for work already tracked unless the user explicitly
picks "Create new".

If there are related but distinct issues (not duplicates), reference them only
when it helps clarify scope, and proceed.

### 3. Resolve Ambiguity

You are the project manager. Your job is to guarantee that what ships matches
what the requester intended — zero deviation. A vague issue ships the wrong
thing, so close the gaps *before* drafting, not in code review. Be ruthless:
err toward asking. An issue that defers a product decision has failed.

After investigating, list every product decision the request leaves open — the
choices that change *what* gets built, not *how* it is coded. Typical sources of
ambiguity:

- **Concrete enumeration.** When the request implies a set ("actions",
  "statuses", "roles", "events"), the actual members are a product decision.
  "Add typed actions" is incomplete until you have the exact action list and
  what each one does. Do not ship the framework and leave the list open.
- **Per-item behavior.** For each member of a set, what exactly is required,
  what happens on success/failure, what values are valid.
- Scope of a setting (per-user vs. per-organization vs. global).
- Which surface/flow it applies to, and edge cases (empty value, defaults).
- Behavior boundaries (what is explicitly in vs. out of scope).
- Conflicts with existing features or issues found during investigation.

**Ask in rounds — loop until clean.** One batch is almost never enough, because
answers cascade: each decision unlocks the next ("define the fields now" → "which
fields?"; "add a second org set" → "which org, and how does it differ?"). Do not
stop after one round.

Run this loop:

1. List every product decision currently open (use the sources above).
2. Batch the open decisions into one `AskUserQuestion` call — concrete options,
   not one question at a time. Skip any you can settle from the codebase or an
   obvious default (state the default you chose).
3. Fold the answers in. Then re-derive: did any answer open a new product
   decision? Did it expose an enumeration or value you still don't have?
4. If yes, run another round. Repeat until a full pass surfaces zero new product
   questions.

**Stop condition (the sniff test):** a competent developer could read the draft
and build it without asking a single product question — no "which ones?", no
"what happens when?", no "what's the value?". Only when a whole pass produces
nothing new do you proceed to draft. If nothing is genuinely ambiguous from the
start, proceed without asking.

Never resolve a cascaded question by guessing (e.g. inventing field names because
asking again felt like too many rounds). Guessing here is the exact failure this
loop prevents — ask the next round instead.

### 4. Draft The Issue

Mirror the dominant existing issue format. Keep it tight and skimmable — bullets
over paragraphs, no filler. If no format exists, use:

```markdown
## Summary

- One line: what changes and why.
- Affected surface(s): `path/to/file`, UI flow, schema — only the useful ones.
- Constraints / decided scope: any boundary the implementer must respect.

## Acceptance Criteria

- [ ] Concrete outcome question, answerable yes/no?
- [ ] Validation or behavior question, answerable yes/no?
```

Good issue bodies:

- Open with a single lead line, then bullets. No multi-sentence paragraphs.
- Name relevant files, commands, APIs, UI surfaces, schemas, or docs only when
  they are useful entry points — one per line.
- State constraints the implementer must preserve as short bullets.
- Fold resolved ambiguities (from step 3) into the constraints/scope bullets so
  the decision is recorded, not re-litigated.
- Separate distinct problems into separate issues unless the user asks for an
  umbrella issue.
- Acceptance criteria test concrete observable behavior ("a file-upload action
  cannot complete without a file"), not whether the framework exists ("actions
  declare a typed requirement"). If a criterion would pass with an empty/stub
  implementation, rewrite it.

Avoid:

- Prose paragraphs, restating obvious context, or words a competent implementer
  does not need. Honor the length caps in Rules.
- Long code archaeology that belongs in a plan, PR review, or implementation
  notes.
- Pseudocode, exact function shapes, or a task checklist that removes the
  implementer's freedom.
- "TBD" / "out of scope" / "maintainer decides later" for any product decision —
  resolve it in step 3 instead.
- Acceptance criteria that are vague, multi-part, or impossible to answer yes/no.

### 5. Create Or Update The Issue

If the user chose to edit an existing issue in step 2, update it in place
instead of creating a new one:

- GitHub: `gh issue edit <n> --body-file <file>` (and `--title` if it changed;
  `gh issue reopen <n>` first if reviving a closed issue).
- GitLab: `glab issue update <n> --description-file <file>`.

Otherwise create the issue in the detected tracker using its normal CLI/API:

- GitHub: `gh issue create --title "<title>" --body-file <file>`
- GitLab: `glab issue create --title "<title>" --description-file <file>`
- Other trackers: use the repository's observed tool or API.

Use a temporary body file when possible so quoting and markdown are preserved.
If creating multiple issues, create them one at a time unless the tracker and
environment make parallel creation reliable, then verify the final issue numbers
and URLs.

### 6. Verify And Report

After creation, read the created issue back from the tracker and confirm:

- The title and body were saved correctly.
- The body follows the repo's issue format.
- Acceptance criteria are the final section and contain 1-4 checkboxed yes/no
  questions.

Report the issue number, title, and URL. Mention duplicate or access caveats
only when present.
