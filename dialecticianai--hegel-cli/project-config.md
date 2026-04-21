---
trigger: always_on
description: **Hegel** orchestrates Dialectic-Driven Development through state-based workflows. Use it for structured development cycles, command guardrails, AST-aware code transformations, and metrics collection.
---

# Using Hegel for Workflow Orchestration

**Hegel** orchestrates Dialectic-Driven Development through state-based workflows. Use it for structured development cycles, command guardrails, AST-aware code transformations, and metrics collection.

**Core principle:** Use when structure helps, skip when it doesn't. The user always knows best.

---

## Command Reference

All commands support `--help` for detailed options. Use `hegel <command> --help` for specifics.

**State directory override:** All commands accept `--state-dir <path>` flag or `HEGEL_STATE_DIR` env var to override default `.hegel/` location. Useful for testing, multi-project workflows, or CI/CD.

### Initialization

```bash
hegel init          # Smart detection: greenfield or retrofit workflow
hegel config list   # View all configuration
hegel config get <key>
hegel config set <key> <value>
```

**Config keys:**
- `code_map_style` - `monolithic` or `hierarchical` (default: hierarchical)
- `use_reflect_gui` - Auto-launch review GUI: `true` or `false` (default: true)

**Init workflows:**
- **Greenfield** (no code): Creates CLAUDE.md, VISION.md, ARCHITECTURE.md, initializes git
- **Retrofit** (existing code): Analyzes structure, creates code maps in README.md files, integrates DDD patterns

### Meta-Modes & Workflows

```bash
hegel meta <learning|standard>  # Declare meta-mode (optional)
hegel meta                      # View current meta-mode

hegel start <workflow> [node]   # Load workflow (optionally at specific node)
hegel status                    # Show current state
hegel next                      # Advance to next phase (auto-infers completion claim)
hegel done                      # Advance and assert reaches 'done' phase (error if not)
hegel restart                   # Return to SPEC phase (restart cycle, keep same workflow)
hegel repeat                    # Re-display current prompt
hegel abort                     # Abandon workflow entirely (required before starting new one)
hegel reset                     # Clear all state
```

**Meta-modes:**
- `learning` - Research ↔ Discovery loop (starts with research)
- `standard` - Discovery ↔ Execution (starts with discovery)

**Workflows:**
- `cowboy` - **DEFAULT** - Minimal overhead for straightforward tasks (just LEXICON guidance)
- `init-greenfield` - CUSTOMIZE_CLAUDE → VISION → ARCHITECTURE → GIT_INIT (new projects)
- `init-retrofit` - DETECT_EXISTING → CODE_MAP → CUSTOMIZE_CLAUDE → VISION → ARCHITECTURE → GIT_COMMIT (existing projects)
- `research` - PLAN → STUDY → ASSESS → QUESTIONS (external knowledge gathering)
- `discovery` - SPEC → PLAN → CODE → LEARNINGS → README (toy experiments)
- `execution` - Production-grade rigor with code review phase
- `refactor` - Focused refactoring workflow

**Starting at custom nodes:**
```bash
# Start at default beginning
hegel start discovery           # Starts at 'spec' node

# Start at specific node (skip earlier phases)
hegel start discovery plan      # Start directly at plan phase
hegel start execution code      # Start directly at code phase
```

**Custom start nodes are useful for:**
- Resuming interrupted workflows
- Testing specific workflow phases
- Skipping phases you've already completed manually

**What happens:**
- `hegel start` prints the first phase prompt with embedded guidance
- `hegel start <workflow> <node>` starts at specified node (validates node exists)
- `hegel next` advances and prints the next phase prompt - **follow these instructions**
- `hegel repeat` re-displays current prompt if you need to see it again
- `hegel restart` returns to SPEC phase (same workflow, fresh cycle)
- `hegel abort` abandons workflow entirely (required before starting different workflow)

**Guardrails:**
- Cannot start new workflow while one is active → run `hegel abort` first
- Invalid start node returns error with list of available nodes
- Prevents accidental loss of workflow progress

### Workflow Stashing

Save and restore workflow snapshots for context switching:

```bash
hegel stash save -m "message"   # Save current workflow with message
hegel stash save                # Save without message
hegel stash list                # List all stashes (newest first)
hegel stash pop [index]         # Restore stash (defaults to 0) and delete it
hegel stash drop [index]        # Delete stash without restoring
```

**Format:** `stash@{0}: execution/code "message" (2 hours ago)`

**Common workflow:**
```bash
# Save current work
hegel stash save -m "feature A in progress"

# Work on something else
hegel start discovery
# ... do work ...
hegel abort

# Resume feature A
hegel stash pop
```

**Guardrails:**
- Cannot stash if no active workflow
- Cannot pop if workflow is active (must `abort` or `stash` first)
- Stashes stored in `.hegel/stashes/` with auto-indexing

### Code Operations

```bash
hegel astq [options] [path]     # AST-based search/transform (wraps ast-grep)
```

**Critical:** Use `hegel astq --help` for pattern syntax and examples. ALWAYS prefer astq over grep/rg for code search (AST-aware, ignores comments/strings, explicit "no matches" feedback).

### Document Review

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dialecticianai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
