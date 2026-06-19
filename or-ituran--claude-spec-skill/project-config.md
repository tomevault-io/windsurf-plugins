---
trigger: always_on
description: This skill implements a **checkpoint-driven specification workflow** where:
---

---
name: spec
description: This skill should be used when the user asks to "create a spec", "write specification", "spec out a feature", "design a feature", "document requirements", or when planning features before implementation. Also auto-invokes when .spec folder exists and user discusses feature design.
version: 1.0.0
---

# Specification Creator with Iterative Checkpoints

This skill implements a **checkpoint-driven specification workflow** where:
- **Every step is checkpointed** before asking the user to continue
- **User controls the pace** - can stop and resume at any checkpoint
- **Main orchestrator stays lean** - all heavy work delegated to sub-agents
- **Context is preserved** in files, not in conversation
- **Output feeds into TDD** - acceptance criteria ready for test-driven implementation

## Core Principles

### 1. Checkpoint-First Design
```
Step → Save State → Show Result → Ask "Continue?" → Wait for User
```

### 2. Sub-Agent Delegation
The orchestrator ONLY:
- Coordinates workflow
- Saves/loads checkpoints
- Asks user questions
- Invokes sub-agents via Task tool

The orchestrator NEVER:
- Reads large amounts of code directly
- Performs complex analysis inline
- Writes spec content directly

### 3. Lean Context
All exploration, analysis, and writing is done by sub-agents. Results are summarized back to the orchestrator.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              Spec Orchestrator (Main Conversation)              │
│  - Checkpoint management                                        │
│  - User interaction (AskUserQuestion)                           │
│  - Phase transitions                                            │
│  - Sub-agent invocation via Task tool                           │
└──────────────────────────────┬──────────────────────────────────┘
                               │ Task tool calls
       ┌───────────────────────┼───────────────────────┐
       │                       │                       │
       ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  DISCOVERY      │   │  DESIGN         │   │  FINALIZE       │
│ spec-context-   │   │ spec-functional │   │ spec-document-  │
│   analyzer      │   │   -designer     │   │   writer        │
│ spec-domain-    │   │ spec-api-       │   │ spec-validator  │
│   explorer      │   │   designer      │   │ spec-acceptance │
│ spec-constraint │   │ spec-edge-case  │   │   -generator    │
│   -gatherer     │   │   -finder       │   └─────────────────┘
└─────────────────┘   └─────────────────┘
```

## Checkpoint Types

| Checkpoint | When | State Saved |
|------------|------|-------------|
| `SESSION_CREATED` | After session initialization | Session folder, initial input |
| `CONTEXT_ANALYZED` | After codebase analysis | Related code, patterns, architecture |
| `DOMAIN_EXPLORED` | After domain exploration | Entities, relationships, terminology |
| `CONSTRAINTS_GATHERED` | After constraint gathering | NFRs, dependencies, limitations |
| `FUNCTIONAL_DESIGNED` | After functional design | Requirements, user stories |
| `API_DESIGNED` | After interface design | Contracts, data models |
| `EDGE_CASES_IDENTIFIED` | After edge case analysis | Boundaries, error scenarios |
| `SPEC_DRAFTED` | After document creation | Complete spec draft |
| `SPEC_VALIDATED` | After validation | Feasibility confirmed |
| `ACCEPTANCE_GENERATED` | After criteria generation | Testable acceptance criteria |
| `SESSION_COMPLETE` | After finalization | Archived, ready for TDD |

## Workflow: Step by Step

### Phase 0: Session Detection

**Orchestrator Action**: Check for existing sessions

```bash
# Check for active sessions
ls .spec/sessions/*/context.json 2>/dev/null
```

**If session exists**:
- Show session summary
- Ask: "Resume this session or start new?"

**If no session**:
- Proceed to Phase 1

---

### Phase 1: Initialize Session

#### Step 1.1: Create Session Structure

**Orchestrator Action**: Create folders and initial files

```bash
# Run init script
bash ~/.claude/skills/spec/scripts/init-spec-folder.sh "feature-name"
```

**Checkpoint**: `SESSION_CREATED`
- Save: session_id, created_at
- Ask user for initial input via AskUserQuestion:
  - What feature/change are you specifying?
  - Any known requirements or constraints?
  - Target components/areas of the codebase?

Save responses to `input.md`

---

### Phase 2: Discovery

#### Step 2.1: Context Analysis

**Orchestrator Action**: Invoke sub-agent

```
Task(
  subagent_type="spec-context-analyzer",
  prompt="Analyze the codebase for context related to: [feature description]
          Target areas: [from user input]

          Save results to: [session_path]/discovery/context.md

          Return a brief summary (5-10 lines) of:
          - Related existing code and patterns found
          - Architecture constraints identified
          - Key integration points"
)
```

**Checkpoint**: `CONTEXT_ANALYZED`
- Save: context_summary in context.json
- Show user: Summary from agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [or-ituran/claude-spec-skill](https://github.com/or-ituran/claude-spec-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
