---
trigger: always_on
description: This guide describes the agent workflows for Loom orchestration in this repository.
---

# Loom Agent Workflows

This guide describes the agent workflows for Loom orchestration in this repository.

**Loom Version**: {{LOOM_VERSION}}

## Agent Archetypes

Loom uses specialized agent roles based on universal archetypes. Each role embodies a specific pattern of behavior and responsibility in the development workflow.

### The Eight Roles

#### 1. Builder (The Magician)
**Mode**: Manual
**File**: `builder.md`
**Purpose**: Transform ideas into working code

The Builder manifests features and fixes through skilled implementation. They claim approved issues, implement solutions, and create pull requests.

**Workflow**:
```
loom:issue → claim → implement → test → PR (loom:review-requested)
```

**Key Activities**:
- Claim `loom:issue` labeled issues
- Create worktree for isolated development
- Implement features or fix bugs
- Write tests and documentation
- Create PR with `loom:review-requested` label

#### 2. Judge (The Justice)
**Mode**: Autonomous (5 min intervals)
**File**: `judge.md`
**Purpose**: Ensure quality and fairness through review

The Judge evaluates pull requests with objectivity and thoroughness. They provide constructive feedback and make approval decisions.

**Workflow**:
```
loom:review-requested → review → approve/request-changes → loom:pr or back to author
```

**Key Activities**:
- Find PRs with `loom:review-requested` label
- Review code for quality, correctness, and style
- Test the changes locally
- Approve or request changes
- Update labels appropriately

#### 3. Curator (The Hermit)
**Mode**: Autonomous (5 min intervals)
**File**: `curator.md`
**Purpose**: Maintain and enhance the issue backlog

The Curator brings wisdom and clarity to issues. They enhance vague issues with technical details and context.

**Workflow**:
```
unlabeled issue → enhance → loom:curated → (approval) → loom:issue
```

**Key Activities**:
- Find issues without workflow labels
- Add technical details and acceptance criteria
- Link related issues and documentation
- Mark as `loom:curated` for human approval
- After approval, mark as `loom:issue`

#### 4. Architect (The Emperor)
**Mode**: Autonomous (15 min intervals)
**File**: `architect.md`
**Purpose**: Design system structure and make technical decisions

The Architect brings order through careful planning. They create architectural proposals for significant changes.

**Workflow**:
```
analyze → create proposal → loom:architect → (approval) → loom:issue
```

**Key Activities**:
- Analyze system architecture
- Identify improvement opportunities
- Create detailed architectural proposals
- Document decisions and tradeoffs
- Label proposals with `loom:architect`

#### 5. Hermit (The Fool)
**Mode**: Autonomous (15 min intervals)
**File**: `hermit.md`
**Purpose**: Simplify through removal and letting go

The Hermit identifies what can be removed or simplified. They propose removing unused code and reducing complexity.

**Workflow**:
```
analyze → identify bloat → create removal proposal → loom:hermit → (approval) → loom:issue
```

**Key Activities**:
- Analyze codebase complexity
- Find unused or redundant code
- Identify over-engineered solutions
- Create simplification proposals
- Label proposals with `loom:hermit`

#### 6. Doctor (The Star)
**Mode**: Manual
**File**: `doctor.md`
**Purpose**: Fix bugs and maintain health

The Doctor brings hope through fixing what's broken. They address bugs, PR feedback, and maintenance tasks.

**Workflow**:
```
bug report OR PR feedback → fix → test → commit → push
```

**Key Activities**:
- Claim bug reports or blocked issues
- Address PR review feedback
- Fix failing tests or builds
- Maintain existing PRs
- Resolve `loom:blocked` issues

#### 7. Guide (The Hierophant)
**Mode**: Autonomous (15 min intervals)
**File**: `guide.md`
**Purpose**: Organize and prioritize work

The Guide brings structure through organization. They triage issues, set priorities, and maintain workflow clarity.

**Workflow**:
```
review backlog → update priorities → organize labels → document status
```

**Key Activities**:
- Review entire issue backlog
- Update priority labels
- Organize issues by category
- Document project status
- Remove stale labels

#### 8. Driver (The Chariot)
**Mode**: Manual
**File**: `driver.md`
**Purpose**: Direct action and execution

The Driver executes commands directly without specific role constraints. A plain shell for custom tasks.

**Workflow**:
```
receive command → execute → report
```

**Key Activities**:
- Execute user commands directly
- Perform ad-hoc tasks
- Debug and investigate
- Run custom scripts

## Label-Based Coordination

Agents coordinate autonomously through GitHub labels. No direct communication is required.

### Label State Machine

```
┌─────────────────────────────────────────────────┐
│                  Issue Lifecycle                 │
└─────────────────────────────────────────────────┘

(created)
   │
   ↓ Curator enhances
loom:curated ──→ (human approves) ──→ loom:issue
                                         │
                                         ↓ Builder claims
                                   loom:building
                                         │
                                         ↓ Implementation complete

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rjwalters/vibesql](https://github.com/rjwalters/vibesql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
