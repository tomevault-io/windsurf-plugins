---
trigger: always_on
description: **CRITICAL: Check this FIRST on every session:**
---

# Instructions for Claude Code Sessions

## Verify Launch Directory

**CRITICAL: Check this FIRST on every session:**

```bash
# Verify we're in the community-patterns repository root
git remote get-url origin 2>/dev/null | grep -q "community-patterns"
```

**If this fails:**

- User launched Claude from the WRONG directory
- **STOP IMMEDIATELY and tell the user:**
  - "Please quit Claude and relaunch it from your community-patterns directory"
  - "You can find it wherever you cloned it (e.g.,
    `cd ~/Code/community-patterns` or
    `cd ~/Code/common-tools/community-patterns`)"
  - "Then run `claude` from there"

**If this succeeds:**

- Continue with setup checks below

---

## First-Time Setup Check

**Check if this is first-time setup:**

```bash
# Quick check: Does workspace config exist?
test -f .claude-workspace && echo "Setup complete" || echo "First-time setup needed"
```

**If "First-time setup needed":**

- This is **FIRST-TIME SETUP**
- **STOP HERE and run GETTING_STARTED.md**
- Follow that guide step-by-step to set up:
  - labs repository
  - .env file with API keys
  - upstream remote
  - user's workspace
  - workspace config file (.claude-workspace)
  - first pattern

**If "Setup complete":**

- User is already set up
- Load workspace config (see Step 2)
- Continue with Session Startup Sequence below

---

## Session Startup Sequence

**At the start of every session, use the `session-startup` skill.**

The session-startup skill handles:

- Checking for upstream updates in this repo
- Updating labs and patterns repositories
- Loading workspace configuration
- Checking and starting dev servers if needed

This is critical to do at the start of every session to get latest instructions
and ensure the environment is ready.

---

## ⚠️ WHEN STUCK - READ THIS FIRST

**CRITICAL: If you encounter ANY of these situations, IMMEDIATELY use the
`community-docs` skill:**

- ❓ Something doesn't work as expected
- ❓ An error you don't understand
- ❓ Unsure how to implement something
- ❓ A pattern that should work but doesn't
- ❓ Behavior that seems like a bug
- ❓ Confused about framework conventions
- ❓ Tried something twice and it failed both times

**DO NOT spin your wheels. Community docs contain hard-won knowledge from other
developers who hit the same issues.**

```
STUCK? → Use `community-docs` skill IMMEDIATELY
        ↓
Still stuck? → Check official labs docs
        ↓
Still stuck after 1-2 attempts? → Use `strategic-investigation` skill
        ↓                         (enter plan mode, launch parallel subagents)
Still stuck after investigation? → Use `recovery-strategies` skill (Steps 3-4)
        ↓
Still stuck? → Ask user for help
```

**After 1-2 failed attempts:** Don't keep guessing. Use the
`strategic-investigation` skill to:

- Enter plan mode and step back from implementation
- Launch parallel Explore agents to find idiomatic solutions
- Understand WHY the solution is correct before implementing
- Find the right approach, not just "anything that works"

**The community-docs skill is your FIRST LINE OF DEFENSE when anything goes
wrong.**

---

## Repository Structure

```
community-patterns/        # THIS REPO (user's fork or direct)
├── .claude-workspace      # Workspace config: username, is_fork, setup status (gitignored)
├── CLAUDE.md              # This file - Claude's instructions
├── GETTING_STARTED.md     # First-time setup guide (Claude-guided)
├── DEVELOPMENT.md         # Normal development workflows
├── README.md              # Quick overview with warnings
├── SETUP.md               # Setup instructions
└── patterns/
    ├── examples/          # Shared examples (READ-ONLY)
    ├── alice/, bob/, ...  # Other users (READ-ONLY)
    └── $GITHUB_USER/      # USER's workspace (WRITABLE)
        ├── README.md      # Optional: user's notes
        ├── WIP/           # Work-in-progress patterns
        │   └── *.tsx      # Patterns under active development
        └── *.tsx          # Stable/production patterns

~/Code/labs/               # Framework repo (separate, READ-ONLY)
```

### User Workspace Structure

**Recommended organization within `patterns/$GITHUB_USER/`:**

**WIP/** - Work in progress

- **IMPORTANT: Most pattern development should happen in WIP/**
- Patterns actively being developed
- Experimental features
- Not fully tested
- Can be messy/incomplete
- Keep working here until pattern is stable and tested

**Importing from labs** - Direct cross-repo imports

- Import patterns directly from `../../../labs/packages/patterns/...`
- The `--root` flag is auto-injected by `scripts/cf` for cross-repo resolution
- No need to copy files locally — always use the canonical version in labs

**Root level** - Stable patterns

- Only move patterns here when fully tested and working
- Completed, tested patterns
- Ready for use or sharing
- Well-documented

**design/todo/** - Development documentation

- TODO files for complex patterns
- Track design decisions, progress, and context
- Named to match pattern files (e.g., `pattern-name.md`)
- Permanent documentation checked into git
- See "TODO Files as Working Memory" section

**issues/** - Framework questions and architecture issues

- Document complex framework problems
- Questions for framework authors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkomoros/community-patterns](https://github.com/jkomoros/community-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
