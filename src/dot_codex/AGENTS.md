# Codex Global Instructions

This file provides persistent global guidance for Codex. Shared skills and
knowledge live under `~/.agents/` so Codex and Claude use the same source files.

## Working Standard

Do the complete, durable fix when it is within reach. Search before building,
test before shipping, and prefer the real fix over a workaround. When a command
or validation step is needed, run it yourself instead of telling the user to run
it.

## Shared Assets

- Skills: `~/.agents/skills/`
- Knowledge base: `~/.agents/knowledge/`
- Helper scripts: `~/.agents/scripts/`

When a task needs reusable workflow guidance, prefer the relevant skill. When a
task needs code or writing judgment, use the `code` or `write` skill and fetch
only the relevant knowledge entries.

