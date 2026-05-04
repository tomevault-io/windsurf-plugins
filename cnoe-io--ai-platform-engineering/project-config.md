---
trigger: always_on
description: **Project**: AI Platform Engineering
---

# AI Platform Engineering Development Rules

## Repository Information

**Project**: AI Platform Engineering
**Type**: Python Backend for AI Agents & Multi-Agent System
**Language**: Python 3.11+
**Framework**: LangGraph, LangChain, A2A Protocol
**Package Manager**: uv
**Testing**: pytest

## Git Commit Standards

### Conventional Commits (REQUIRED)

All commits MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Commit Types

- **feat**: A new feature
  ```
  feat: add ArgoCD MCP pagination support
  feat(supervisor): implement TODO-based execution plan
  feat(agent): add OOM protection for large queries
  ```

- **fix**: A bug fix
  ```
  fix: resolve A2A artifact streaming race condition
  fix(argocd): handle 819 applications without OOM
  fix(streaming): prevent duplicate artifact warnings
  ```

- **docs**: Documentation only changes
  ```
  docs: add ADR for OOM protection strategy
  docs(adr): document MCP pagination implementation
  ```

- **perf**: Performance improvements
  ```
  perf(argocd): optimize pagination for large datasets
  perf(mcp): reduce memory usage in list operations
  ```

- **refactor**: Code change that neither fixes a bug nor adds a feature
  ```
  refactor(agent): simplify context window management
  ```

- **test**: Adding missing tests or correcting existing tests
  ```
  test: add integration tests for pagination
  ```

- **build**: Changes to build system or dependencies
  ```
  build: update langchain to v0.2.0
  build(docker): optimize multi-agent containers
  ```

- **ci**: Changes to CI configuration files and scripts
  ```
  ci: add GitHub Actions for integration tests
  ```

- **chore**: Other changes that don't modify src or test files
  ```
  chore: update .gitignore
  ```

#### Breaking Changes

Breaking changes MUST be indicated with `!` after the type/scope:

```
feat!: change A2A artifact format

BREAKING CHANGE: Artifact names must now use explicit types
(tool_notification_start, execution_plan_update, etc.)
```

### Developer Certificate of Origin (DCO) - REQUIRED

Every commit MUST include a DCO sign-off:

```
Signed-off-by: Your Name <your.email@example.com>
```

#### How to Sign Off

**Option 1: Command Line (Recommended)**
```bash
git commit -s -m "feat: add new feature"
```

**Option 2: Configure Git to always sign off**
```bash
git config --global format.signoff true
```

#### DCO Meaning

By signing off, you certify that:
- You wrote the code or have the right to submit it
- You understand the code will be distributed under the project's license
- You agree to the Developer Certificate of Origin v1.1

### Complete Commit Example

```
feat(argocd): implement MCP pagination for list operations

Added strict pagination to all ArgoCD MCP list operations to prevent
OOM issues caused by large responses (e.g., 819 applications).

Changes:
- Added page and page_size parameters (default 20, max 100)
- Implemented pagination metadata in responses
- Added safety limits for search operations
- Updated agent prompts to handle paginated results

Prevents memory exhaustion from loading entire datasets.

Closes #789

Signed-off-by: Your Name <your.email@example.com>
```

## Documentation: Spec Kit vs ADRs

This project uses **two complementary documentation systems**:

| System | Location | Purpose | Lifecycle |
|--------|----------|---------|-----------|
| **Spec Kit** | `.specify/specs/` | Planning & tracking active work | Living document, updated as work progresses |
| **ADRs** | `docs/docs/changes/` | Decision rationale for posterity | Immutable once decision is made |

### When to Use Spec Kit (Primary)

**Spec Kit is the PRIMARY documentation for most changes.**

✅ **Create a Spec for:**
- New features (agents, MCP tools, UI features)
- Bug fixes with implementation phases
- Performance improvements
- Multi-agent orchestration changes
- Any work with acceptance criteria to track

Specs track: WHAT we're building, HOW we're building it, STATUS of each phase

### When to Use ADRs (Significant Decisions Only)

**ADRs are for significant architectural decisions that need permanent historical record.**

✅ **Create an ADR ONLY when:**
- The decision has **lasting architectural impact**
- Multiple alternatives were considered with **significant tradeoffs**
- Future developers will ask **"why did we do it this way?"**
- The decision affects **multiple components or teams**

Examples requiring ADRs:
- Choosing A2A protocol over alternatives
- Adopting counter-based vs event-based streaming
- Selecting pagination strategy for OOM protection

❌ **Don't create ADR for:**
- Standard bug fixes (use Spec Kit instead)
- Minor optimizations
- Dependency updates
- Configuration changes

### ADR Location

**ALL ADRs MUST be created in**: `docs/docs/changes/`

```
ai-platform-engineering/
├── docs/
│   └── docs/
│       └── changes/            ← ADRs go here
│           ├── YYYY-MM-DD-title.md
```

### Decision Flow

```
New Work Item
     │
     ▼
┌────────────────────┐
│ Create Spec in     │  ← ALWAYS do this first
│ .specify/specs/    │
└────────────────────┘
     │
     ▼
┌────────────────────────────────────┐
│ Does this involve a significant    │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cnoe-io/ai-platform-engineering](https://github.com/cnoe-io/ai-platform-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
