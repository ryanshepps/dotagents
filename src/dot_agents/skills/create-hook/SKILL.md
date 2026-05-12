---
name: create-hook
description: Design, create, and test Claude or Codex hooks. Use when the user wants automation around tool use, prompt submission, stop events, validation, formatting, or safety checks.
---

# Create Hook

Create practical hooks for Claude, Codex, or both.

## Flow

1. Detect the target runtime: Claude, Codex, or both. If unclear, ask once.
2. Inspect existing hook config:
   - Claude: `~/.claude/settings.json` or `src/dot_claude/settings.json.tmpl`.
   - Codex: `~/.codex/hooks.json`, `~/.codex/config.toml`, or
     `src/dot_codex/hooks.json.tmpl`.
3. Determine event, matcher, scope, and blocking behavior.
4. Write the hook script in the runtime-specific hook directory or a shared
   script location when both runtimes can use it.
5. Register the hook in the correct config file.
6. Test with representative JSON input before considering it done.

## Runtime Notes

Claude and Codex hook schemas are similar but not identical. Do not blindly copy
config between them. Shared shell scripts should tolerate both input shapes where
possible, especially `tool_input.file_path`, `tool_input.filePath`, and
Codex's `tool_name`.

For Codex, enable hooks with:

```toml
[features]
hooks = true
```

For Claude, keep Go-template-specific config in `src/dot_claude/settings.json.tmpl`.
