---
trigger: always_on
description: <!-- SW:META template="agents" version="1.0.314" sections="index,quickstart,rules,orchestration,principles,commands,nonclaudetools,syncworkflow,contextloading,structure,agents,skills,taskformat,usformat,workflows,plugincommands,troubleshooting,docs" -->
---

<!-- SW:META template="agents" version="1.0.314" sections="index,quickstart,rules,orchestration,principles,commands,nonclaudetools,syncworkflow,contextloading,structure,agents,skills,taskformat,usformat,workflows,plugincommands,troubleshooting,docs" -->

<!-- SW:SECTION:index version="1.0.314" -->
## Section Index (Use Ctrl+F to Navigate)

| Section | Search For | Purpose |
|---------|------------|---------|
| Rules | `#essential-rules` | Critical rules, file organization |
| **Orchestration** | `#workflow-orchestration` | **Plan Mode, Subagents, Verification** |
| **Principles** | `#core-principles` | **Quality: Simplicity, No Laziness** |
| Commands | `#commands` | All SpecWeave commands |
| **Hooks** | `#non-claude-tools` | **CRITICAL: Hook behavior to mimic** |
| **User Story** | `#user-story-format` | **CRITICAL: Project/Board fields** |
| Sync | `#sync-workflow` | When/how to sync |
| Context | `#context-loading` | Efficient context loading |
| Troubleshoot | `#troubleshooting` | Common issues |
<!-- SW:END:index -->

<!-- SW:SECTION:quickstart version="1.0.314" -->
## Quick Start

1. **Get Project Context FIRST**: `specweave context projects` (save the output!)
2. **Create Your First Increment**: `/sw:increment "your-feature"`
3. **Customize**: Edit spec.md - **EVERY User Story needs `**Project**:` field!**
4. **Execute**: `/sw:do` to start implementation
<!-- SW:END:quickstart -->

<!-- SW:SECTION:rules version="1.0.314" -->
## Essential Rules {#essential-rules}

```
1. NEVER pollute project root with .md files
2. Increment IDs unique (0001-9999)
3. ⛔ ONLY 4 files in increment root: metadata.json, spec.md, plan.md, tasks.md
4. ⛔ ALL reports/scripts/logs → increment subfolders (NEVER at root!)
5. metadata.json MUST exist BEFORE spec.md can be created
6. tasks.md + spec.md = SOURCE OF TRUTH (update after every task!)
7. ⛔ EVERY User Story MUST have **Project**: field
8. ⛔ For 2-level structures: EVERY US also needs **Board**: field
```

### ⛔ INCREMENT FOLDER CLEANLINESS (CRITICAL!)

**Increment folders MUST stay organized. NEVER create random files at increment root!**

| File Type | Correct Location |
|-----------|-----------------|
| Reports, summaries, analysis (*.md) | `reports/` |
| Validation/QA/completion reports | `reports/` |
| Auto-session summaries | `reports/` |
| Logs, execution output | `logs/{YYYY-MM-DD}/` |
| Helper scripts | `scripts/` |
| Domain docs | `docs/domain/` |

**File Organization**:
```
# ✅ CORRECT - clean increment structure
.specweave/increments/0001-feature/
├── metadata.json                  # REQUIRED - create FIRST
├── spec.md                        # WHAT & WHY
├── plan.md                        # HOW (optional)
├── tasks.md                       # Task checklist
├── reports/                       # ALL other .md files go here!
│   ├── validation-report.md
│   ├── completion-report.md
│   └── auto-session-summary.md
├── scripts/                       # Helper scripts
└── logs/                          # Execution logs
    └── 2026-01-04/

# ❌ WRONG - polluted increment folder!
.specweave/increments/0001-feature/
├── metadata.json
├── spec.md
├── tasks.md
├── completion-report.md          # WRONG! Move to reports/
├── auto-session-summary.md       # WRONG! Move to reports/
└── some-analysis.md              # WRONG! Move to reports/
```
<!-- SW:END:rules -->

<!-- SW:SECTION:orchestration version="1.0.314" -->
## Workflow Orchestration {#workflow-orchestration}

**Claude Code has built-in orchestration features. Non-Claude tools must implement these manually.**

### 1. Plan Mode Default (Use SpecWeave Increments!)

**Claude Code**: Has `EnterPlanMode` tool → triggers `/sw:increment` workflow automatically.

**Non-Claude Tools - Use SpecWeave Increment Structure:**
```
BEFORE implementing ANY non-trivial task (3+ steps):

1. STOP - Don't start coding immediately
2. Create increment folder: `.specweave/increments/XXXX-feature/`
3. Create the 3 required files:
   - spec.md   → WHAT & WHY (user stories, acceptance criteria)
   - plan.md   → HOW (architecture, approach, risks)
   - tasks.md  → Task checklist with test plans
4. GET USER APPROVAL before implementing

If something goes sideways during implementation:
→ STOP and re-plan (don't keep pushing)
→ Update spec.md/plan.md with revised approach
→ Get approval again if scope changed
```

**SpecWeave Planning Files:**

**spec.md** (WHAT & WHY):
```markdown
---
increment: 0001-feature-name
title: "Feature Title"
---

### US-001: User Story Title
**Project**: my-app              # ← MANDATORY! Get from: specweave context projects

**As a** [user type]
**I want** [goal]
**So that** [benefit]

**Acceptance Criteria**:
- [ ] **AC-US1-01**: [Criterion 1]
- [ ] **AC-US1-02**: [Criterion 2]
```

**plan.md** (HOW):
```markdown
# Plan: Feature Name

## Approach
[High-level architecture/approach]

## Risks & Decisions
- [ ] Decision: [question needing user input]
- Risk: [potential issue and mitigation]
```

**tasks.md** (Checklist):
```markdown
### T-001: Task Title
**User Story**: US-001
**Satisfies ACs**: AC-US1-01

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tristancgardner/macos-dictate](https://github.com/tristancgardner/macos-dictate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
