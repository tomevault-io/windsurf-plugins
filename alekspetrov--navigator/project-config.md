---
trigger: always_on
description: Sessions that last. AI that learns. Features that ship.
---

# Navigator: Finish What You Start

Sessions that last. AI that learns. Features that ship.

## Why This Exists

**The problem**: AI coding sessions crash at exchange 5-7. Context window fills with documentation you never use.

**Navigator's solution**: Context engineering—load what you need, when you need it. 150k → 12k tokens (92% reduction).

**Result**: Sessions go 20+ exchanges. Features actually ship.

**Proven**: OpenTelemetry-verified, not estimates. Session efficiency scores 94/100.

**NEW in v5.0.0**: Theory of Mind integration based on Riedl & Weidmann 2025 research - bilateral modeling, quality detection, and ToM verification checkpoints.

**NEW in v5.1.0**: Loop Mode with structured completion signals, dual-condition exit gates, and stagnation detection - inspired by Ralph's autonomous loop framework.

**NEW in v5.4.0**: Code Simplification - automatic code clarity improvements before commit. Based on Anthropic's internal code-simplifier pattern. Clarity over brevity, functionality preserved absolutely.

**NEW in v5.5.0**: Auto-Update on Session Start - automatically updates Navigator when newer version detected. No manual `nav-upgrade` needed for daily releases.

**NEW in v5.6.0**: Task Mode - unified workflow orchestration that coordinates between skills, loop mode, and direct execution. Auto-detects complexity and defers to appropriate handler.

**NEW in v5.7.0**: Feature Management - view and toggle Navigator features via `nav-features` skill. Shows feature table on first session after install/update.

**NEW in v5.8.0**: Auto-Update Project Sync - auto-update now syncs project config after plugin update. Version drift detection warns when project config is behind. Restart prompt after mid-session updates.

**NEW in v5.9.0**: Workflow Enforcement - mandatory WORKFLOW CHECK block before task responses. Loop Mode and Task Mode triggers auto-detected. Complexity scoring. Hook-based enforcement available.

**NEW in v6.0.0**: Project Knowledge Graph - unified search across tasks, SOPs, system docs, and experiential memories. Patterns, pitfalls, decisions, and learnings persist across sessions. Query with "What do we know about X?". Auto-surfaces relevant memories on session start.

**NEW in v6.1.0**: Multi-Agent Production - parallel Claude agents with visual dashboard. Natural language trigger: "Run multi-agent workflow for TASK-XX". Role templates (orchestrator, implementer, tester, reviewer, documenter) with minimal context (~5k each). Real-time terminal dashboard shows progress. 3x faster than sequential.

---

## How You'll Use It

**Every session starts with**:
```
"Start my Navigator session"
```

This loads:
- Navigator index (2k tokens) - what exists
- Current task context (3k tokens) - what you're working on
- Nothing else (yet)

**As you work**:
- Need system architecture? Loads on-demand (5k)
- Need SOP? Loads when relevant (2k)
- Need integration details? Loads if required

**Result**: Context window stays efficient. Sessions last 20+ exchanges without restart.

---

## Understanding Context Efficiency

**New to this approach?** Read the philosophy:
- `.agent/philosophy/CONTEXT-EFFICIENCY.md` - Why Navigator exists
- `.agent/philosophy/ANTI-PATTERNS.md` - Common mistakes (upfront loading, etc.)
- `.agent/philosophy/PATTERNS.md` - What works and why

**Quick start?** Continue to [Navigator Workflow](#navigator-workflow-critical---enforce-strictly)

---

## Core Principle

**Context engineering beats bulk loading.**

Not "load everything just in case."
Not "better safe than sorry."

Strategic loading saves 92% of context for actual work.

---

## Navigator Workflow (CRITICAL - ENFORCE STRICTLY)

### WORKFLOW ENFORCEMENT (MANDATORY - READ FIRST)

**Before responding to ANY task request, you MUST show this block:**

```
┌─────────────────────────────────────┐
│ WORKFLOW CHECK                      │
├─────────────────────────────────────┤
│ Loop trigger: [YES/NO]              │
│ Complexity: [0.X]                   │
│ Mode: [LOOP/TASK/DIRECT]            │
└─────────────────────────────────────┘
```

**Loop Mode triggers** (if ANY match → Mode: LOOP):
- "run until done", "do all", "keep going", "iterate until"
- "finish this", "complete everything", "don't stop"

**Task Mode triggers** (if complexity >= 0.5 → Mode: TASK):
- Multi-file changes, refactoring, new features
- Planning required, architecture changes
- "implement", "refactor", "add feature", "fix all"

**Direct execution** (if neither → Mode: DIRECT):
- Single file edit, quick fix, simple question
- Complexity < 0.5, no loop trigger

**If you skip this check, you are violating Navigator workflow.**

### SESSION START PROTOCOL (MANDATORY)

**🚨 EVERY new conversation/session MUST begin with**:

```
"Start my Navigator session"
```
OR (legacy): `/nav:start`

**What this does**:
1. Checks for Navigator updates (auto-updates if enabled)
2. Loads `.agent/DEVELOPMENT-README.md` (navigator)
3. Loads user profile for bilateral modeling (if exists)
4. Checks for assigned tasks from PM tool (if configured)
5. Sets Navigator workflow context
6. Activates token optimization strategy
7. Applies ToM preferences from profile

**If user doesn't start session**:
- You MUST proactively suggest it
- Never proceed without loading navigator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alekspetrov/navigator](https://github.com/alekspetrov/navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
