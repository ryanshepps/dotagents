---
name: analyze-usage
description: Analyze local Claude and Codex usage patterns and recommend what should become skills, plugins, agents, hooks, or persistent instructions.
---

# Analyze Usage

Analyze local agent usage and produce practical refactoring recommendations.

## Scope

Inspect available local session/history data for Claude and Codex. Common
locations include:

- Claude: `~/.claude/`
- Codex: `~/.codex/`
- Shared skills and knowledge: `~/.agents/`

Respect sandbox limits and do not exfiltrate private transcript content. Report
patterns and short examples only when needed.

## Output

Group findings into:

- What the user does most frequently.
- What should become skills.
- What should become plugins.
- What should become agents.
- What belongs in `CLAUDE.md`, `AGENTS.md`, or shared repo guidance.
- What should become hooks or validation scripts.

Prefer concrete file paths and candidate names over abstract advice.

