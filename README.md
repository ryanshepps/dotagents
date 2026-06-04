# dotagents

Personal Claude and Codex configuration, managed by
[chezmoi](https://www.chezmoi.io/).

## Layout

```text
.chezmoiroot                          -> points chezmoi at src/
CLAUDE.md                             -> Claude project instructions
AGENTS.md                             -> Codex project instructions
README.md                             -> this file
src/
  .chezmoi.toml.tmpl                  -> init template; prompts for work bool
  dot_agents/                         -> materializes to ~/.agents/
    skills/                           -> shared skills, each with SKILL.md
    knowledge/                        -> shared code/write knowledge bases
    scripts/                          -> shared helper and validation scripts
  dot_claude/                         -> materializes to ~/.claude/
    commands/                         -> Claude slash commands
    hooks/executable_*.sh             -> Claude-compatible hook scripts
    settings.json.tmpl                -> Claude settings
    symlink_*                         -> ~/.claude compatibility links
  dot_codex/                          -> materializes to ~/.codex/
    AGENTS.md                         -> Codex global instructions
    config.toml                       -> Codex global config
    hooks.json.tmpl                   -> Codex hook config
```

## Runtime Shape

Shared assets live once under `~/.agents/`:

```text
~/.agents/skills/
~/.agents/knowledge/
~/.agents/scripts/
```

Claude compatibility is provided by symlinks:

```text
~/.claude/skills    -> ~/.agents/skills
~/.claude/knowledge -> ~/.agents/knowledge
~/.claude/scripts   -> ~/.agents/scripts
```

Codex reads skills directly from `~/.agents/skills` and its global instructions
from `~/.codex/AGENTS.md`.

## Applying

All chezmoi commands for this repo use the dedicated config:

```bash
alias ccm='chezmoi --config=$HOME/.config/chezmoi/dotagents.toml'
```

Then:

- `ccm apply` syncs `~/.agents/`, `~/.claude/`, and `~/.codex/`.
- `ccm diff` previews pending changes.
- `ccm status` verifies runtime state.

## Knowledge Management

The shared knowledge base has two domain-scoped corpora:

- `src/dot_agents/knowledge/code/` for software-engineering laws, style rules,
  language guides, and workflow patterns.
- `src/dot_agents/knowledge/write/` for prose craft, tone, structure, and
  format rules.

Each leaf is a markdown file with frontmatter:

```yaml
---
slug: <slug>
categories: [<category>]
priority: 1
description: <one sentence>
applies_when:
  - <task context>
related: []
---
```

Regenerate and validate MOCs with:

```bash
python3 src/dot_agents/scripts/gen_mocs.py --knowledge-dir src/dot_agents/knowledge/code
python3 src/dot_agents/scripts/validate_kb.py --knowledge-dir src/dot_agents/knowledge/code
```

Use the same commands with `write` for the writing KB.

## Compatibility Rules

- Add shared skills to `src/dot_agents/skills/<name>/SKILL.md`.
- Add shared knowledge to `src/dot_agents/knowledge/<domain>/`.
- Add Claude-only slash commands to `src/dot_claude/commands/`.
- Every Claude command must have a matching shared skill with the same name so
  Codex has equivalent behavior.
- Add Codex-only config to `src/dot_codex/`, not `src/dot_claude/`.
- Run `python3 src/dot_agents/scripts/validate_dual_agent_repo.py` after
  structural changes.
