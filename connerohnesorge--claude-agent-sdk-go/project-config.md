---
trigger: always_on
description: <!-- spectr:START -->
---

<!-- spectr:START -->
# Spectr Instructions

Instructions for AI coding assistants using Spectr for spec-driven development.

## TL;DR Quick Checklist

- Search existing work: `spectr spec list --long`, `spectr list` (use `rg` only for full-text search)
- Decide scope: new capability vs modify existing capability
- Pick a unique `change-id`: kebab-case, verb-led (`add-`, `update-`, `remove-`, `refactor-`)
- Scaffold: `proposal.md`, `tasks.md`, `design.md` (only if needed), and delta specs per affected capability
- Write deltas: use `## ADDED|MODIFIED|REMOVED|RENAMED Requirements`; include at least one `#### Scenario:` per requirement
- Validate: `spectr validate [change-id] --strict` and fix issues
- Request approval: Do not start implementation until proposal is approved

## Three-Stage Workflow

### Stage 1: Creating Changes
Create proposal when you need to:
- Add features or functionality
- Make breaking changes (API, schema)
- Change architecture or patterns
- Optimize performance (changes behavior)
- Update security patterns

Triggers (examples):
- "Help me create a change proposal"
- "Help me plan a change"
- "Help me create a proposal"
- "I want to create a spec proposal"
- "I want to create a spec"

Loose matching guidance:
- Contains one of: `proposal`, `change`, `spec`
- With one of: `create`, `plan`, `make`, `start`, `help`

Skip proposal for:
- Bug fixes (restore intended behavior)
- Typos, formatting, comments
- Dependency updates (non-breaking)
- Configuration changes
- Tests for existing behavior

**Workflow**
1. Review `spectr/project.md`, `spectr list`, and `spectr list --specs` to understand current context.
2. Choose a unique verb-led `change-id` and scaffold `proposal.md`, `tasks.md`, optional `design.md`, and spec deltas under `spectr/changes/<id>/`.
3. Draft spec deltas using `## ADDED|MODIFIED|REMOVED Requirements` with at least one `#### Scenario:` per requirement.
4. Run `spectr validate <id> --strict` and resolve any issues before sharing the proposal.

### Stage 2: Implementing Changes
Track these steps as TODOs and complete them one by one.
1. **Read proposal.md** - Understand what's being built
2. **Read design.md** (if exists) - Review technical decisions
3. **Read tasks.md** - Get implementation checklist
4. **Implement tasks sequentially** - Complete in order
5. **Confirm completion** - Ensure every item in `tasks.md` is finished before updating statuses
6. **Update checklist** - After all work is done, set every task to `- [x]` so the list reflects reality
7. **Approval gate** - Do not start implementation until the proposal is reviewed and approved

### Stage 3: Archiving Changes
After deployment, create separate PR to:
- Move `changes/[name]/` → `changes/archive/YYYY-MM-DD-[name]/`
- Update `specs/` if capabilities changed
- Use `spectr archive <change-id> --skip-specs --yes` for tooling-only changes (always pass the change ID explicitly)
- Run `spectr validate --strict` to confirm the archived change passes checks

## Before Any Task

**Context Checklist:**
- [ ] Read relevant specs in `specs/[capability]/spec.md`
- [ ] Check pending changes in `changes/` for conflicts
- [ ] Read `spectr/project.md` for conventions
- [ ] Run `spectr list` to see active changes
- [ ] Run `spectr list --specs` to see existing capabilities

**Before Creating Specs:**
- Always check if capability already exists
- Prefer modifying existing specs over creating duplicates
- Use `spectr show [spec]` to review current state
- If request is ambiguous, ask 1–2 clarifying questions before scaffolding

### Search Guidance
- Enumerate specs: `spectr spec list --long` (or `--json` for scripts)
- Enumerate changes: `spectr list` (or `spectr change list --json` - deprecated but available)
- Show details:
  - Spec: `spectr show <spec-id> --type spec` (use `--json` for filters)
  - Change: `spectr show <change-id> --json --deltas-only`
- Full-text search (use ripgrep): `rg -n "Requirement:|Scenario:" spectr/specs`

## Quick Start

### CLI Commands

```bash
# Essential commands
spectr list                  # List active changes
spectr list --specs          # List specifications
spectr show [item]           # Display change or spec
spectr validate [item]       # Validate changes or specs
spectr archive <change-id> [--yes|-y]   # Archive after deployment (add --yes for non-interactive runs)

# Project management
spectr init [path]           # Initialize Spectr
spectr update [path]         # Update instruction files

# Interactive mode
spectr show                  # Prompts for selection
spectr validate              # Bulk validation mode

# Debugging
spectr show [change] --json --deltas-only
spectr validate [change] --strict
```

### Command Flags

- `--json` - Machine-readable output
- `--type change|spec` - Disambiguate items
- `--strict` - Comprehensive validation
- `--no-interactive` - Disable prompts
- `--skip-specs` - Archive without spec updates
- `--yes`/`-y` - Skip confirmation prompts (non-interactive archive)

## Directory Structure

```
spectr/
├── project.md              # Project conventions
├── specs/                  # Current truth - what IS built
│   └── [capability]/       # Single focused capability
│       ├── spec.md         # Requirements and scenarios
│       └── design.md       # Technical patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/connerohnesorge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
