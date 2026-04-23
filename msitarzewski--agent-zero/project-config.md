---
trigger: always_on
description: **Version**: 2.2 (2025-03-04) | **Compatibility**: Claude, Cursor, Copilot, Cline, Aider, all AGENTS.md-compatible tools
---

# AGENTS.md

**Version**: 2.2 (2025-03-04) | **Compatibility**: Claude, Cursor, Copilot, Cline, Aider, all AGENTS.md-compatible tools
**Status**: Canonical single-file guide for AI-assisted development

---

## Table of Contents

1. [Compliance & Core Rules](#1-compliance--core-rules)
2. [Session Startup](#2-session-startup)
   - [Compaction Protocol](#compaction-protocol-mid-session-context-preservation)
3. [Memory Bank](#3-memory-bank)
4. [State Machine](#4-state-machine)
5. [Task Contract & Budgets](#5-task-contract--budgets)
6. [Quality & Documentation](#6-quality--documentation)
7. [Example Workflow](#7-example-workflow)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Compliance & Core Rules

### Startup Compliance (Output Every Session)

```
COMPLIANCE CONFIRMED: Reuse over creation

⚠️  GIGO PREVENTION - User Responsibilities:
📋 Clear task objectives | 🔗 Historical context | 🎯 Success criteria
⚙️  Architectural constraints | 🎖️ You lead - clear input = excellent output

[Continue with Memory Bank loading...]
```

### The Four Sacred Rules

| Rule | Requirement | Validation |
|------|-------------|------------|
| ❌ **No new files without reuse analysis** | Search codebase, reference files that cannot be extended, provide exhaustive justification | Before creating: "Analyzed X, Y, Z. Cannot extend because [technical reason]" |
| ❌ **No rewrites when refactoring possible** | Prefer incremental improvements, justify why refactoring won't work | "Refactoring X impossible because [specific limitation]" |
| ❌ **No generic advice** | Cite `file:line`, show concrete integration points, include migration strategies | Every suggestion includes `file:line` citation |
| ❌ **No ignoring existing architecture** | Load patterns before changes, extend existing services/components, consolidate duplicates | "Extends existing pattern at `file:line`" |

### Reuse Validation Checklist (Before Creating Files)

```markdown
- [ ] Searched: [search terms] → found: [list files]
- [ ] Analyzed extension:
  - [ ] `existing/file1.ext` - Cannot extend: [specific technical reason]
  - [ ] `existing/file2.ext` - Cannot extend: [specific technical reason]
- [ ] Checked patterns: `systemPatterns.md#[section]`
- [ ] Justification: New file needed because [exhaustive reasoning]
```

### Non-Negotiables

- **Approval Gates**: No file changes without explicit user approval
- **Citations**: Always `file:line` for code, `file.md#Section` for Memory Bank
- **Sandbox First**: All edits in branch/temp clone, never main
- **MCP Preferred**: Use MCP servers for memory, repo ops, QA over brute-force context
- **No Mock Data**: Never fake/simulated data in production; never stub functions
- **Context Engineering**: Keep working context focused on current task

---

## 2. Session Startup

### Load Priority (Choose Based on Task Complexity)

**Every Session** (mandatory):
1. Output compliance statement (Section 1)
2. Attach MCP servers: Read `.brain/mcp.config.json` or `.mcp.json` if present
3. Load Memory Bank per mode below
4. Log session: `{"ts":"2025-10-25T10:30Z","mode":"fast|standard|deep","mb_v":"2024-10"}`

**Fast Track** (bug fixes, small changes):
```
- [ ] Load current month README: `memory-bank/tasks/YYYY-MM/README.md`
- [ ] Check recent achievements and next priorities
- [ ] Load `quick-start.md` if needed
```

**Standard Discovery** (features, tests, quality-critical work):
```
- [ ] Current month README
- [ ] Core files: projectbrief.md, systemPatterns.md, techContext.md, activeContext.md, progress.md
- [ ] Scan docs/ for recent updates
- [ ] Scan root for instructions.md, ai_instructions.md
- [ ] Verify toc.md and activeContext.md current
```

**Deep Dive** (architecture, legacy investigation):
```
- [ ] Standard Discovery files
- [ ] Specific month README when investigating legacy
- [ ] decisions.md for architectural context
- [ ] Cross-reference with current work patterns
```

### Session Logging (Operational Log - Separate from Memory Bank)

Append-only JSONL format:
```json
{"timestamp":"2025-10-25T10:30:00Z","session_id":"uuid","mode":"standard","mb_version":"2024-10"}
{"timestamp":"2025-10-25T10:35:00Z","session_id":"uuid","event":"state_transition","from":"PLAN","to":"BUILD"}
{"timestamp":"2025-10-25T11:00:00Z","session_id":"uuid","event":"approval_requested","state":"APPROVAL"}
```

### Compaction Protocol (Mid-Session Context Preservation)

Compaction (context compression) can happen at any time — triggered by the system automatically, by the user via `/compact`, or by platform-level context management. **The agent does not control compaction timing and may not get advance notice.** Therefore, state persistence must be continuous, not deferred to a pre-compaction moment.

#### Continuous State Persistence (At Every State Transition)

At each state transition (`PLAN → BUILD → DIFF → QA → APPROVAL → APPLY → DOCS`), persist the following to the Memory Bank:

1. **State machine position**: Update `activeContext.md` with current state, substate, and working context
2. **Task progress**: Append current status to `tasks/YYYY-MM/README.md` with `[IN-PROGRESS]` tag
3. **Decisions**: Append any new architectural decisions to `decisions.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msitarzewski/AGENT-ZERO](https://github.com/msitarzewski/AGENT-ZERO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
