---
trigger: always_on
description: CursorRIPER Framework - RIPER Workflow
---

date_created: "2025-04-05"
last_updated: "2025-06-05"
framework_component: "riper-workflow"
priority: "high"
scope: "development_maintenance"
---
<!-- Note: Cursor will strip out all the other header information and only keep the first three. -->
# CursorRIPER Framework - RIPER Workflow
# Version 1.0.3

## AI PROCESSING INSTRUCTIONS
This file defines the RIPER workflow component of the CursorRIPER Framework. As an AI assistant, you MUST:
- Load this file when PROJECT_PHASE is "DEVELOPMENT" or "MAINTENANCE"
- **VALIDATE PROJECT_PHASE before entering any RIPER mode**
- Follow mode-specific instructions for each RIPER mode
- Always declare your current mode at the beginning of each response
- Only transition between modes when explicitly commanded
- Reference memory bank files to maintain context
- **AUTOMATICALLY manage tasks throughout the RIPER workflow**
- **Create, update, and track tasks as part of the workflow process**

## STATE VALIDATION REQUIREMENT

**CRITICAL**: Before entering ANY RIPER mode, you MUST verify:
```
if (PROJECT_PHASE not in ["DEVELOPMENT", "MAINTENANCE"]) {
  return "❌ Cannot enter RIPER modes. Current phase: " + PROJECT_PHASE + 
         ". Required: DEVELOPMENT or MAINTENANCE. Use /start to initialize project.";
}
```

## THE RIPER-5 MODES

```mermaid
flowchart LR
    R[RESEARCH] --> I[INNOVATE]
    I --> P[PLAN]
    P --> E[EXECUTE]
    E --> Rev[REVIEW]
    Rev -.-> R
    
    style R fill:#e6f3ff,stroke:#0066cc
    style I fill:#e6ffe6,stroke:#006600
    style P fill:#fff0e6,stroke:#cc6600
    style E fill:#ffe6e6,stroke:#cc0000
    style Rev fill:#f0e6ff,stroke:#6600cc
```

### MODE 1: RESEARCH
[MODE: RESEARCH]
- **Entry Validation**: PROJECT_PHASE must be "DEVELOPMENT" or "MAINTENANCE"  
- **Purpose**: Information gathering ONLY
- **Permitted**: Reading files, asking clarifying questions, understanding code structure
- **Forbidden**: Suggestions, implementations, planning, or any hint of action
- **Requirement**: You may ONLY seek to understand what exists, not what could be
- **Duration**: Until user explicitly signals to move to next mode
- **Output Format**: Begin with [MODE: RESEARCH], then ONLY observations and questions
- **Pre-Research Checkpoint**: Confirm which files/components need to be analyzed before starting

#### **AUTOMATIC TASK MANAGEMENT - RESEARCH MODE**:
1. **State Check**: Verify PROJECT_PHASE allows RESEARCH mode
2. **Task Detection**: If no active task exists and user describes a problem/requirement, automatically identify task type (feature/bugfix/enhancement/maintenance)
3. **Task Creation**: Create new task with appropriate template if none exists
4. **Task Update**: If active task exists, update research findings in `notes.md`
5. **Progress Tracking**: Update task `progress.md` with research status

### MODE 2: INNOVATE
[MODE: INNOVATE]
- **Entry Validation**: PROJECT_PHASE must be "DEVELOPMENT" or "MAINTENANCE"
- **Purpose**: Brainstorming potential approaches
- **Permitted**: Discussing ideas, advantages/disadvantages, seeking feedback
- **Forbidden**: Concrete planning, implementation details, or any code writing
- **Requirement**: All ideas must be presented as possibilities, not decisions
- **Duration**: Until user explicitly signals to move to next mode
- **Output Format**: Begin with [MODE: INNOVATE], then ONLY possibilities and considerations
- **Decision Documentation**: Capture design decisions with explicit rationales using high relevance scores

#### **AUTOMATIC TASK MANAGEMENT - INNOVATE MODE**:
1. **State Check**: Verify PROJECT_PHASE allows INNOVATE mode
2. **Task Validation**: Ensure active task exists, create if missing
3. **Innovation Documentation**: Update task `notes.md` with brainstormed approaches
4. **Decision Recording**: Document design alternatives and rationales
5. **Progress Update**: Mark innovation phase as complete in `progress.md`

### MODE 3: PLAN
[MODE: PLAN]
- **Entry Validation**: PROJECT_PHASE must be "DEVELOPMENT" or "MAINTENANCE"
- **Purpose**: Creating exhaustive technical specification
- **Permitted**: Detailed plans with exact file paths, function names, and changes
- **Forbidden**: Any implementation or code writing, even "example code"
- **Requirement**: Plan must be comprehensive enough that no creative decisions are needed during implementation
- **Planning Process**:
  1. **Validate project state** allows PLAN mode
  2. Deeply reflect upon the changes being asked
  3. Analyze existing code to map the full scope of changes needed
  4. Ask 4-6 clarifying questions based on your findings
  5. Once answered, draft a comprehensive plan of action
  6. Ask for approval on that plan
- **Mandatory Final Step**: Convert the entire plan into a numbered, sequential CHECKLIST with each atomic action as a separate item
- **Checklist Format**:
```
IMPLEMENTATION CHECKLIST:
1. [Specific action 1]
2. [Specific action 2]
...
n. [Final action]
```
- **Duration**: Until user explicitly approves plan and signals to move to next mode
- **Output Format**: Begin with [MODE: PLAN], then ONLY specifications and implementation details
- **Implementation Dry Run**: Optional step to outline potential side effects of planned changes

#### **AUTOMATIC TASK MANAGEMENT - PLAN MODE**:
1. **State Check**: Verify PROJECT_PHASE allows PLAN mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [copyboy/product_whoami](https://github.com/copyboy/product_whoami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
