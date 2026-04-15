---
trigger: always_on
description: This project contains SND (Something Need Doing) Lua macros for FFXIV automation using Dalamud plugins.
---

# SND Macro Development Guidelines

This project contains SND (Something Need Doing) Lua macros for FFXIV automation using Dalamud plugins.

## Development Rules

### Rule #1: Update Skills on Discovery
**Whenever we discover something new about the SND API, fix a bug, or learn the correct way to do something, IMMEDIATELY update the relevant skill documentation in `.claude/skills/`.**

This includes:
- API corrections (e.g., `Player.Level` returns nil, use `Svc.ClientState.LocalPlayer.Level`)
- New patterns that work
- Gotchas and pitfalls to avoid
- Working code examples from testing

The skills are our living documentation - keep them accurate!

### Rule #2: Update Documentation Before Push
**Whenever you change functionality in a script, you MUST update the script's documentation (the comment block at the top) before pushing to SND.**

This includes:
- Version number bump (MAJOR.MINOR.PATCH)
- Description updates if behavior changed
- HOW IT WORKS section if flow changed
- Any new features or changed behavior documented

Never push functional changes without updating the docs first!

### Rule #3: Verify Config Flags Affect All Relevant Code Paths
**When adding or modifying config flags/options, verify they affect ALL places in the code where they should apply.**

Before pushing:
1. Search for all usages of the feature the flag controls
2. Ensure every code path respects the flag
3. Check edge cases (e.g., nested function calls, early returns)
4. If a function returns a value that signals special behavior (like "custom macro ran"), ensure all callers handle it

Example: If adding `CustomEndMacro` that runs instead of `/ice start`:
- Find ALL places that call `StartIce()`
- Make sure each call handles the case where custom macro was run
- Return appropriate signals so the script exits cleanly

### Rule #4: Generate Discord Questions for Unknown Issues
**When debugging fails repeatedly and you identify something you don't know about the SND API, generate a copy/paste ready question for Discord.**

Format the question like this:
```
📋 **SND API Question**

**What I'm trying to do:**
[Brief description]

**Code that's failing:**
```lua
[relevant code snippet]
```

**Error message:**
[exact error if available]

**What I've tried:**
- [attempt 1]
- [attempt 2]

**Question:**
[Specific question about the API]
```

This helps get accurate answers from the SND community when documentation is unclear.

### Rule #5: "push snd" or "snd push" Command
**When the user says "push snd" or "snd push", sync ALL local scripts to SND using the node sync tool.**

Run:
```bash
node sync.js push
```

This pushes all Lua scripts from the local Playroom folder to SND's internal storage (the "Synced" folder in SND).

### Rule #6: Always Push to SND After Script Changes
**After making any changes to Lua scripts, ALWAYS run `node sync.js push` to sync to SND without asking.**

Don't ask "Want me to push?" - just push automatically after any script modification.

### Rule #7: Use GitHub MCP for Git Operations
**Use the GitHub MCP tools instead of bash git commands whenever possible.**

Available MCP tools:
- `mcp__github__push_files` - Push multiple files in one commit
- `mcp__github__create_or_update_file` - Create/update a single file
- `mcp__github__list_commits` - View commit history
- `mcp__github__get_file_contents` - Read files from repo
- `mcp__github__create_pull_request` - Create PRs
- `mcp__github__create_branch` - Create branches

Use these instead of `git add`, `git commit`, `git push`, etc.

### Rule #8: Skills Are Plugin-Centric, Not Macro-Centric
**Skills should be organized by PLUGIN, not by macro or content type.**

When discovering new APIs:
- Create/update skills named after the **plugin** (e.g., `snd-autohook`, `snd-visland`, `snd-simpletweaks`)
- Do NOT create skills for specific macros or content (e.g., ~~`snd-ocean-fishing`~~)
- This allows skills to be reused across multiple macros that use the same plugin

Example - From FishingRaid.lua we discovered:
- AutoHook APIs → add to `snd-autohook` skill
- Visland APIs → add to `snd-visland` skill
- SimpleTweaks commands → add to `snd-simpletweaks` skill
- SND built-in ocean fishing functions → add to `snd-core` skill

This modular approach means any future macro can leverage the same plugin documentation.

### Rule #9: Include Source Repository in Skills
**When creating or updating skills from official plugin repositories, ALWAYS include the source repo URL in the skill documentation.**

Add a "Source Status" note near the top of the skill file:
```markdown
> **Source:** https://github.com/Owner/RepoName/path/to/IPC.cs
```

Or if unverified (from screenshots/testing):
```markdown
> **Source Status:** Unverified - documented from screenshots and testing. Official repo: https://github.com/Owner/RepoName
```

This allows us to:
- Easily find the official source for updates
- Verify documentation accuracy
- Track which skills need official verification

---

## Skills Documentation

Detailed documentation is organized into modular skills in `.claude/skills/`:

### Core Skills
| Skill | Description |
|-------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vaoan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
