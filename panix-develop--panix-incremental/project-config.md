---
trigger: always_on
description: When working on tasks, automatically read and follow these files based on context:
---

# Copilot Instructions

## Workflow Files

When working on tasks, automatically read and follow these files based on context:

### Task Management
Before starting any work, check for these files and follow their instructions:

1. `.tasks/task-list-[feature-name].md` - Current task list, always check this first
2. `.tasks/prd-[feature-name].md` - Product requirements document

### Workflow Rules
- When I say "next task" → Read `.ai/process-task-list.md` and follow it, if there is a next sub-task in the current task list, if all prior sub-tasks are complete, move the task list and the prd to `.ai/tasks/completed/`
- When I say "create prd" → Read `.ai/create-prd.md` and follow it, the generated prd should be saved to `.tasks/prd-[feature-name].md`
- When I say "generate tasks from prd" → Read `.ai/generate-tasks.md` and follow it, the generated task list should be saved to `.tasks/task-list-[feature-name].md`

### Git Branch Strategy

#### Phase-Based Development
- **Create a phase branch at the start of each phase** using format: `phase/phase[N]`
  - Example: `phase/phase1`, `phase/phase2`, `phase/phase3`
  - This branch serves as the integration branch for all tasks within that phase
  
- **Create task branches for individual sub-tasks** using format: `task/phase[N]-task[X.Y]-[short-description]`
  - Example: `task/phase1-task1.1-fix-scrolling`, `task/phase2-task3.2-add-settings-ui`
  - Always branch FROM the current phase branch, not from main
  
#### Workflow for Each Task
1. **Start task:** Create task branch from phase branch
   ```bash
   git checkout phase/phase1
   git checkout -b task/phase1-task1.1-fix-scrolling
   ```
2. **Complete work:** Make changes, verify no errors
3. **Before committing:**
   - Run `npm run test` or equivalent to verify NO ERRORS in code
   - Verify at least 70% of tests pass (if tests exist)
   - Check with `get_errors` tool to ensure no linting/compilation errors
   - Only commit when code is error-free and tests pass
4. **Commit:** Use conventional commit format
   - `feat(scope): description` for features
   - `fix(scope): description` for bugfixes
   - `test(scope): description` for tests
   - `refactor(scope): description` for refactoring
5. **Merge back:** Merge task branch into phase branch
   ```bash
   git checkout phase/phase1
   git merge task/phase1-task1.1-fix-scrolling --no-ff
   ```
6. **Stay on phase branch** for next task

#### After Completing Parent Task (e.g., Task 1.0 with all sub-tasks)
- **Provide overview of completed work:**
  - List all sub-tasks completed
  - Summarize changes made
  - List files modified
  - Confirm all tests pass and no errors

#### After Completing Entire Phase
- **Wait for user approval** ("phase complete", "ready to release", etc.)
- **Merge to main and create release:**
  ```bash
  git checkout main
  git merge phase/phase1 --no-ff
  git push origin main
  git tag v[X.Y.Z] -m "Phase [N] Complete: [Description]"
  git push origin v[X.Y.Z]
  ```
- **Never push broken code** - always verify tests and error-free status first

## About This Project
[Describe what your project does]

---

## Tech Stack

### Frontend
- React (with hooks, functional components)
- Angular (standalone components preferred)
- TypeScript (strict mode)

### Backend
- Python 3.11+
- FastAPI
- SQLAlchemy (async where possible)
- Pydantic for validation

### Databases
- PostgreSQL / MySQL (via SQLAlchemy ORM)
- MongoDB / other NoSQL databases

---

## Core Principles

### KISS (Keep It Simple, Stupid)
- Write the simplest solution that works
- Avoid clever or overly abstract code
- Prefer readability over brevity
- If code needs extensive comments to explain, simplify it

### YAGNI (You Aren't Gonna Need It)
- Don't implement features "just in case"
- Don't over-engineer for hypothetical future requirements
- Build for current requirements, refactor when needed
- Avoid premature optimization

### DRY (Don't Repeat Yourself)
- Extract repeated code into reusable functions/components
- Use shared utilities for common operations
- BUT: don't over-abstract — duplication is better than wrong abstraction
- Rule of three: refactor after third occurrence

### SRP (Single Responsibility Principle)
- Each function does ONE thing well
- Each class/module has ONE reason to change
- Split large functions into smaller, focused ones
- Separate concerns: data fetching, business logic, presentation

### Additional Principles
- Composition over inheritance: Prefer composing behaviors over deep inheritance hierarchies
- Fail fast: Validate inputs early, throw errors immediately when something is wrong
- Explicit over implicit: Be clear about intent, avoid magic

---

## Naming Conventions

### Python

```python
# Variables and functions: snake_case
user_name = "john"
def get_user_by_id(user_id: int) -> User:
    pass

# Classes: PascalCase
class UserService:
    pass

# Constants: UPPER_SNAKE_CASE
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30

# Private/internal: prefix with underscore
_internal_cache = {}
def _helper_function():
    pass

# Boolean variables: use is_, has_, can_, should_ prefix
is_active = True
has_permission = False
can_edit = True
```

### JavaScript / TypeScript

```typescript
// Variables and functions: camelCase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Panix-Develop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
