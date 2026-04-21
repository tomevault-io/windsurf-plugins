---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md Template

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
This is **[Your Project Name]**—[brief description of what the project does and its purpose].

**Production URLs:**
- Frontend: [your-frontend-url]
- Backend: [your-backend-url]

**Current Status:** [Describe current state: deployed, in development, MVP, etc. List key features/capabilities.]

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in PLANS.md) from design to implementation. Keep EXECPLAN.md updated as it is the source of truth and living document for this project.

## Living Documentation

EXECPLAN.md is the single source of truth for this project. You MUST:

1. **Read EXECPLAN.md at session start** - Always read it first to understand current state
2. **Update Progress section** - After completing any task, add a timestamped entry
3. **Update Surprises and Discoveries** - Document any bugs, unexpected behaviors, or workarounds found
4. **Update Decision Log** - Record any significant implementation decisions with rationale
5. **Update Outcomes and Retrospective** - At major stopping points, summarize what's working, what's not, and lessons learned

Never use temporary todo lists as the primary tracking mechanism. All progress must be recorded in EXECPLAN.md so the next session (or a new conversation) can pick up exactly where we left off.

When resuming work:
1. Read EXECPLAN.md first
2. Check the Progress section for incomplete milestones
3. Check Surprises and Discoveries for known issues
4. Continue from documented stopping point

### Source of Truth

<!-- CUSTOMIZE: List your key documentation files -->
All specifications and design decisions are documented in:
- `VISION.md` — Core requirements, design philosophy, success criteria
- `EXECPLAN.md` — Active milestones, known issues, progress tracking
- `.claude/memory/` — External memory (archived milestones, decisions, schemas, reference)


## Memory System

This project uses a tiered memory system to manage complexity.

### Core Memory (Always Loaded)
- `EXECPLAN.md`: Active state, current milestones, known issues
- `CLAUDE.md`: This file - instructions, conventions, memory access

### External Memory (Read on Demand)
Stored in `.claude/memory/`:

<!-- CUSTOMIZE: Replace with your project's memory files -->
| File | Contains | Read When |
|------|----------|-----------|
| `INDEX.md` | Summary of all memory files | Starting new topic |
| `milestones/[feature-area].md` | Completed milestone details | Debugging that feature |
| `decisions/architecture.md` | Structural choices | Proposing changes |
| `decisions/technology.md` | Stack choices | Evaluating alternatives |
| `decisions/patterns.md` | Implementation patterns | Adding features |
| `schemas/database.md` | Database schema | Database work |
| `schemas/api.md` | API specs | API modifications |
| `schemas/components.md` | UI components | UI work |
| `reference/context.md` | Glossary, structure | Onboarding |
| `reference/retrospective.md` | What worked, lessons | Planning |
| `quality/testing.md` | Test infrastructure, patterns, coverage | Writing tests |

### Starting New Work Protocol

Before implementing a new milestone, PROACTIVELY read relevant memory:

1. **Always read first:**
   - `.claude/memory/INDEX.md` - orient to available memory
   - `decisions/architecture.md` - check architectural constraints
   - `decisions/technology.md` - verify stack choices still apply

2. **Read based on work type:**
   <!-- CUSTOMIZE: Map your feature areas to memory files -->
   - [Feature Area 1] work → `milestones/[feature-area-1].md`
   - [Feature Area 2] work → `milestones/[feature-area-2].md`
   - Database changes → `schemas/database.md`
   - API changes → `schemas/api.md`
   - Testing work → `quality/testing.md`

3. **Check for patterns:**
   - Similar past milestones for implementation patterns
   - Related decisions for constraints and trade-offs
   - Known issues that may affect new work

4. **ALWAYS spawn 3 scout subagents before implementing:**

   **This is mandatory for EVERY milestone.** Spawn 3 parallel subagents to search archived memory before writing any code. Failing to understand the memory system will break the repo.

   | Agent | Focus | Searches | Returns |
   |-------|-------|----------|---------|
   | **Decisions Scout** | Constraints & trade-offs | `decisions/*.md` | Relevant constraints, past rationale |
   | **Patterns Scout** | Implementation precedents | `milestones/*.md` | Similar past work, patterns, gotchas |
   | **Schema Scout** | Data structures & APIs | `schemas/*.md`, `reference/*.md` | Affected tables, endpoints, types |

   **Do NOT skip this step.** The scouts ensure you understand existing patterns before making changes.

### Reactive Retrieval Triggers

Also read external memory when:
1. User asks about completed features
2. You encounter unexpected behavior
3. You're unsure about prior decisions
4. Debugging requires implementation context

### Memory Update Protocol

**This is mandatory for EVERY milestone.**

After completing work:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g-troiani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
