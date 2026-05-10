---
trigger: always_on
description: Used when referred by @@EPIC or @@EPICS or when writing or completing any epic level of work or epic level of capabilities.
---

**Epic Planning & Tracking System**
<!-- AI INSTRUCTIONS: Always follow these rules. Never modify this section. -->

**Purpose**: Plan and track large initiatives using the hierarchy: EPIC GROUP → EPIC → PHASE → STEPS

**Hierarchy Definitions**:
- **EPIC GROUP**: Optional grouping of related epics
- **EPIC**: Large collection of features/capabilities  
- **PHASE**: Major milestone within an epic
- **STEP**: Individual feature within a phase

**AI Usage Rules**:
1. **Epic Creation**: When user requests epic planning, create comprehensive plans here
2. **Architecture Integration**: Always consider architecture.mdc when planning epics and include architectural impact in epic steps
3. **No Execution**: Never execute epic work - that happens in workflow_state.mdc  
4. **Natural Language Processing**: Translate user requests into epic context for workflow integration
5. **Progress Updates**: Update epic status when user reports progress or workflow completes
6. **Epic Search**: Search existing epics when user mentions epic work to set proper workflow context

**Context Limits**: Maximum 3 active epics to maintain AI effectiveness

**EPICS PLANS**
<!-- Stores and tracks the EPIC GROUP, EPIC, PHASE and STEP type of plans.  Only to be modified and used by an AI -->

## EPIC PORTFOLIO STATUS

### Current Portfolio Summary
- **Total Active Epics**: 0 (Max: 3)
- **Completed Epics**: 0
- **Blocked Epics**: 0
- **Last Updated**: [Date when portfolio was last updated]

### Portfolio Limits
**Max Active Epics**: 3 (to maintain AI context effectiveness)
**Auto-Archive**: Completed epics older than 6 months

### Epic Status Legend
- ✅ **COMPLETED**: All phases and steps finished
- 🔄 **IN_PROGRESS**: Currently being worked on
- ⏳ **PLANNED**: Planned but not yet started
- 🚫 **BLOCKED**: Stopped due to dependencies or issues
- ⏸️ **PAUSED**: Temporarily suspended
- ❌ **CANCELLED**: No longer needed
- 📦 **ARCHIVED**: Completed and archived for reference

---

## ACTIVE EPICS

<!-- Active epics will be listed here with progress tracking -->

*No active epics currently planned. Use @EPIC or @EPICS to create new epic plans.*

---

## EPIC PROGRESS TRACKING

<!-- This section tracks progress for each epic with detailed status -->

### Simplified Epic Template
```
### EPIC: [Epic Name]
**Status**: [PLANNED/IN_PROGRESS/BLOCKED/COMPLETED]
**Priority**: [High/Medium/Low]
**Started**: [Start date]
**Target Completion**: [Target date]

#### Goal
[Clear business objective and user value]

#### Success Criteria
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]
- [ ] [Measurable outcome 3]

#### Dependencies & Blockers
- [Current dependencies or blocking issues]

#### PHASE 1: [Phase Name] - [STATUS_ICON] [STATUS]
**Goal**: [Phase objective and deliverables]

**Steps:**
1. **[Step Name]**: [What needs to be built/implemented] - [STATUS_ICON] [COMPLETION_%]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - Architecture impact: [How this affects current architecture or introduces new patterns]
   - AI considerations: [Important context for AI execution]
   - Status: [PLANNED/IN_PROGRESS/COMPLETED] ([completion_%])
   - Started: [Date when work began]
   - Last Updated: [Date of last progress]
   - Completed: [Date when step finished]
   - Notes: [Progress notes, decisions, blockers]

2. **[Step Name]**: [What needs to be built/implemented] - [STATUS_ICON] [COMPLETION_%]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - AI considerations: [Important context for AI execution]
   - Status: [PLANNED/IN_PROGRESS/COMPLETED] ([completion_%])
   - Started: [Date when work began]
   - Last Updated: [Date of last progress]
   - Completed: [Date when step finished]
   - Notes: [Progress notes, decisions, blockers]

3. **[Step Name]**: [What needs to be built/implemented] - [STATUS_ICON] [COMPLETION_%]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - AI considerations: [Important context for AI execution]
   - Status: [PLANNED/IN_PROGRESS/COMPLETED] ([completion_%])
   - Started: [Date when work began]
   - Last Updated: [Date of last progress]
   - Completed: [Date when step finished]
   - Notes: [Progress notes, decisions, blockers]

#### PHASE 2: [Phase Name] - [STATUS_ICON] [STATUS]
**Goal**: [Phase objective and deliverables]

**Steps:**
1. **[Step Name]**: [What needs to be built/implemented]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - AI considerations: [Important context for AI execution]

2. **[Step Name]**: [What needs to be built/implemented]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - AI considerations: [Important context for AI execution]

#### PHASE 3: [Phase Name] - [STATUS_ICON] [STATUS]
**Goal**: [Phase objective and deliverables]

**Steps:**
1. **[Step Name]**: [What needs to be built/implemented]
   - Key requirements: [2-3 critical requirements]
   - Acceptance criteria: [What defines "done"]
   - AI considerations: [Important context for AI execution]

#### Notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fbrbovic/cursor-rule-framework](https://github.com/fbrbovic/cursor-rule-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
