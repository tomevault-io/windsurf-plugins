---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
In this repo, beads is **local-only**: use it for structured issue tracking on your machine, but do not expect a shared Dolt remote.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
```

## GitHub Discussions

This repo has GitHub Discussions enabled (`silverstein/minutes`). Issues are for bugs and feature requests. Discussions are for usage questions, setup help, and community show-and-tell.

**Agent guidelines:**
- When triaging an issue that's really a "how do I...?" question, suggest converting it to a Discussion rather than closing it
- When a user's bug report turns out to be a config/setup issue, answer it and note that Discussions is the better venue for follow-ups
- After shipping a feature or fix, check if any open Q&A discussions are resolved by the change — post a reply pointing to the release
- When writing user-facing error messages or help text, link to Discussions (not Issues) for support: `https://github.com/silverstein/minutes/discussions`
- Don't file Discussions as work items — they're community conversations, not tracked tasks

## Portable Agent Skills (`.agents/skills/minutes/`, `.opencode/skills/`)

This repo maintains skill outputs in **three locations**:

- `.claude/plugins/minutes/` — Claude Code plugin (uses `${CLAUDE_PLUGIN_ROOT}`)
- `.agents/skills/minutes/` — Agent-agnostic mirror for Codex, Gemini, and other agents (uses `$MINUTES_SKILLS_ROOT`)
- `.opencode/skills/` — OpenCode-native mirror (one-level discovery path + matching `.opencode/commands/`)

**What lives where:**
- `SKILL.md` files are mirrored 1:1. Content is identical except for path variables and platform-specific references (e.g., "open in desktop app" in the plugin version becomes a CLI command in the agents version).
- `_runtime/hooks/lib/` contains `minutes-learn.mjs` and `minutes-learn-cli.mjs` — the behavioral learning system. These must stay byte-identical across `.agents/skills/minutes/_runtime/hooks/lib/` and `.opencode/skills/_runtime/hooks/lib/`.
- Bundled scripts (`scripts/tag_apply.py`, `scripts/graph_build.py`, etc.) are mirrored into both portable trees.
- `.opencode/commands/*.md` provides native `/minutes-*` slash commands for OpenCode and is generated from the same canonical skill sources.

**When you modify a skill or runtime hook:**
```bash
# Preferred workflow: edit the canonical source under tooling/skills/sources/<name>/skill.md
# then regenerate every host surface from one place.
cd tooling/skills
npm run build
npm run compile

# Verify generated outputs are current:
npm run compile:dry
npm run check
```

**Why multiple trees?** Claude Code plugins use `${CLAUDE_PLUGIN_ROOT}` and plugin metadata. Codex/Gemini consume the `.agents/skills/minutes/` mirror. OpenCode only auto-discovers `skills/*/SKILL.md` one directory deep and has its own `.opencode/commands/` surface, so it needs a flattened generated tree.

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

## macOS Desktop Identity Rule

For any local desktop work that touches macOS privacy / TCC-sensitive features
(Microphone, Screen Recording, Input Monitoring, Accessibility, call capture,
global hotkeys), do **not** dogfood by repeatedly replacing
`/Applications/Minutes.app` with ad-hoc local rebuilds.

Use the dedicated development app identity instead:

```bash
export MINUTES_DEV_SIGNING_IDENTITY="Developer ID Application: Mathieu Silverstein (63TMLKT8HN)"
./scripts/install-dev-app.sh
```

Canonical dogfood target:

- `~/Applications/Minutes Dev.app`

Why:

- macOS TCC permissions attach to the effective app identity and signature
- ad-hoc local rebuilds of `/Applications/Minutes.app` can trigger repeated or misleading permission prompts
- the signed dev app is the stable local identity for permission-sensitive testing

<!-- BEGIN BEADS INTEGRATION profile:full hash:d4f96305 -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Dolt-backed local history for issue state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silverstein/minutes](https://github.com/silverstein/minutes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
