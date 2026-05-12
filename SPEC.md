# SPEC

## Goal

Keep this chezmoi source compatible with Claude and Codex at the same time.
Shared agent assets must live once, while runtime-specific adapters remain
separate.

## Invariants

- Shared skills, knowledge, and helper scripts live under `src/dot_agents/`.
- Claude-specific files live under `src/dot_claude/`.
- Codex-specific files live under `src/dot_codex/`.
- `~/.claude/skills`, `~/.claude/knowledge`, and `~/.claude/scripts` are
  compatibility symlinks to `~/.agents/*`.
- Root `CLAUDE.md` and `AGENTS.md` must both contain the dual-agent contract.
- Every Claude command in `src/dot_claude/commands/*.md` must have a matching
  shared skill in `src/dot_agents/skills/<command>/SKILL.md`.
- Shared skills and scripts must use `~/.agents/...` paths, not
  `~/.claude/...` paths, except when explicitly documenting runtime adapters.

## Validation

Run:

```bash
python3 src/dot_agents/scripts/validate_dual_agent_repo.py
```

For KB changes, also run:

```bash
python3 src/dot_agents/scripts/validate_kb.py --knowledge-dir src/dot_agents/knowledge/code
python3 src/dot_agents/scripts/validate_kb.py --knowledge-dir src/dot_agents/knowledge/write
```

For generated MOC changes, run `gen_mocs.py` first for the affected domain.

