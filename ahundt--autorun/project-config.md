---
trigger: always_on
description: UV workspace containing 2 Claude Code plugins: **autorun**, **pdf-extractor**.
---

# autorun Marketplace - Claude Code

UV workspace containing 2 Claude Code plugins: **autorun**, **pdf-extractor**.

**For Gemini CLI:** See [GEMINI.md](GEMINI.md) for Gemini-specific installation and configuration.

## Installation (Claude Code)

### From GitHub (Production - Recommended)

```bash
# Install directly via Claude Code plugin system
claude plugin install https://github.com/ahundt/autorun.git

# Verify
claude plugin list  # Should show: cr, pdf-extractor
```

### From Local Clone (Development)

```bash
git clone https://github.com/ahundt/autorun.git && cd autorun

# Option 1: UV (recommended - faster, better dependency management)
uv run python -m plugins.autorun.src.autorun.install --install --force

# Option 2: pip fallback (if UV not available)
pip install -e . && python -m plugins.autorun.src.autorun.install --install --force

# REQUIRED: Install as UV tool for global CLI availability
# This makes 'autorun' and 'claude-session-tools' commands globally available
# which are needed for proper daemon operation and session management
cd plugins/autorun && uv tool install --force --editable .

# Verify installation
claude plugin list  # Should show: cr, pdf-extractor
autorun --status  # Verifies UV tool installation works
```

**Install UV (if needed):**
```bash
# macOS/Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh

# Homebrew:
brew install uv

# Windows:
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Test Installation

```bash
# In Claude Code session:
/ar:st  # Expected: "AutoFile policy: allow-all"
```

## Quick Start

```bash
/ar:go <task>     # Start autonomous execution with three-stage verification
/ar:sos           # Emergency stop
/ar:st            # Show current status
```

## Plugins Overview

| Plugin | Prefix | Purpose |
|--------|--------|---------|
| **autorun** | `/ar:` | Autonomous execution, file policies, safety guards, plan export |
| **pdf-extractor** | `/pdf-extractor:` | Extract text from PDFs (9 backends, GPU support) |

---

## autorun Plugin (v0.10.1)

### Three-Stage Verification System

Ensures thorough task completion through mandatory stages:

| Stage | Purpose | Completion Marker |
|-------|---------|-------------------|
| **Stage 1** | Initial implementation | `AUTORUN_INITIAL_TASKS_COMPLETED` |
| **Stage 2** | Critical evaluation - identify gaps, fix issues | `CRITICALLY_EVALUATING_PREVIOUS_WORK_AND_CONTINUING_TASKS_AS_NEEDED` |
| **Stage 3** | Final verification - all requirements met | `AUTORUN_ALL_TASKS_COMPLETED_AND_VERIFIED_SUCCESSFULLY` |

**Concrete Example:**
```
User: /ar:go Add login form with validation and tests

Stage 1: Implements login form → outputs AUTORUN_INITIAL_TASKS_COMPLETED
Stage 2: Reviews work, finds missing error handling, adds it → CRITICALLY_EVALUATING_PREVIOUS_WORK_AND_CONTINUING_TASKS_AS_NEEDED
Stage 3: Verifies form works, tests pass, error handling complete → AUTORUN_ALL_TASKS_COMPLETED_AND_VERIFIED_SUCCESSFULLY → Session ends
```

Without three-stage: Claude might stop after Stage 1 with incomplete work.

### All Commands

**AutoFile Policy** (controls file creation via PreToolUse hooks):

| Short | Long | Legacy | Description |
|-------|------|--------|-------------|
| `/ar:a` | `/ar:allow` | `/afa` | Allow all file creation |
| `/ar:j` | `/ar:justify` | `/afj` | Require `<AUTOFILE_JUSTIFICATION>` for new files |
| `/ar:f` | `/ar:find` | `/afs` | Modify existing files only (strictest) |
| `/ar:st` | `/ar:status` | `/afst` | Show current policy |

**Autorun Control**:

| Short | Long | Legacy | Description |
|-------|------|--------|-------------|
| `/ar:go <task>` | `/ar:run` | `/autorun` | Start autonomous execution |
| `/ar:gp <task>` | `/ar:proc` | `/autoproc` | Procedural mode with Wait Process |
| `/ar:x` | `/ar:stop` | `/autostop` | Graceful stop |
| `/ar:sos` | `/ar:estop` | `/estop` | Emergency stop |

**Plan Management**:

| Short | Long | Description |
|-------|------|-------------|
| `/ar:pn` | `/ar:plannew` | Create structured plan |
| `/ar:pr` | `/ar:planrefine` | Critique and improve plan |
| `/ar:pu` | `/ar:planupdate` | Update plan with new info |
| `/ar:pp` | `/ar:planprocess` | Execute plan with methodology |

**Documentation**:

| Short | Long | Description |
|-------|------|-------------|
| `/ar:gc` | `/ar:commit` | Git commit requirements (17 steps) |
| `/ar:ph` | `/ar:philosophy` | System design philosophy (17 principles) |

**Safety Guards** (v0.6.0+) - Blocks dangerous commands and suggests safe alternatives:

Built-in protections for: `rm` → `trash`, `git reset --hard` → `git stash`, `git clean -f` → `git clean -n`, etc.

| Command | Description |
|---------|-------------|
| `/ar:no <pattern>` | Block command pattern in this session |
| `/ar:ok <pattern> [N\|5m\|perm]` | Allow pattern — `3` uses, `5m` duration, or `perm` (rest of session); default 1 use then auto-revokes |
| `/ar:clear` | Clear all session blocks and allows |
| `/ar:blocks` | Show active session-level blocks and allows |
| `/ar:globalno <pattern>` | Block command pattern globally (persists across sessions) |
| `/ar:globalok <pattern> [N\|5m\|perm]` | Allow pattern globally — `3` uses, `5m` duration, or `perm` (until cleared); default 1 use then auto-revokes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahundt/autorun](https://github.com/ahundt/autorun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
