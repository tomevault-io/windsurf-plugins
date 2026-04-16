---
trigger: always_on
description: This file contains project-specific instructions for AI assistants working on the Apicurio Registry VSCode Extension.
---

# Apicurio VSCode Plugin - Claude Code Instructions

This file contains project-specific instructions for AI assistants working on the Apicurio Registry VSCode Extension.

## Project Overview

**What**: VSCode extension for browsing and editing API specifications from Apicurio Registry
**Tech Stack**: TypeScript, VSCode Extension API, Axios, Model Context Protocol (MCP)
**Current Phase**: Active development - implementing core features

## Documentation Structure

### Critical Documentation Files

1. **`docs/TODO.md`** - Daily quick reference (what to work on today)
2. **`docs/MASTER_PLAN.md`** - Strategic overview and milestones
3. **`docs/tasks/[status]/[priority]/XXX-name.md`** - Individual task specifications
4. **`docs/ai-integration/`** - AI/MCP integration planning and testing

### Documentation Locations

- **Project root**: `/apicurio-vscode-plugin/`
- **Source code**: `src/`
- **Tests**: `test/` or `src/test/suite/`
- **Documentation**: `docs/`
- **Planning**: `docs/planning/`
- **AI Integration**: `docs/ai-integration/`
- **Testing guides**: `docs/testing/`
- **Test data**: `test-data/`

## CRITICAL RULE: Documentation Management

**Documentation is NOT optional. It's part of completing the task.**

### Always Update Documentation When:

1. ✅ Completing a task
2. ✅ Starting a new task
3. ✅ Making architectural decisions
4. ✅ Discovering issues or blockers
5. ✅ Changing priorities
6. ✅ Weekly reviews

### Documentation Update Workflow

#### At Task Completion - MANDATORY

**STOP and ask user:** "Task XXX complete. Let's update documentation together."

**Update checklist:**
- [ ] Move task file: `tasks/in-progress/` → `tasks/completed/`
- [ ] Add "Lessons Learned" section to task file
- [ ] Update `docs/TODO.md`:
  - Move to "Recently Completed" table
  - Update progress percentages
  - Update "What to Work on TODAY"
  - Add entry to "Recent Activity Log"
- [ ] Update `docs/MASTER_PLAN.md` if milestone reached
- [ ] Confirm all updates before proceeding to next task

#### At Task Start

**Ask user:** "Starting Task XXX. Should we update TODO.md?"

**Actions:**
- Update TODO.md "What to Work on TODAY"
- Move task file to `in-progress/` if needed
- Update progress bars

### Quality Check Before Task Completion

**Task is NOT complete until:**
- [ ] Task file moved to completed/
- [ ] TODO.md updated with completion
- [ ] Progress percentages recalculated
- [ ] Activity log entry added
- [ ] MASTER_PLAN.md updated if milestone reached
- [ ] Lessons learned documented
- [ ] User confirms all updates

## Git Workflow - MANDATORY

### Branch Strategy

**ALWAYS use feature branches. NEVER commit directly to main.**

#### Branch Naming Convention

```
task/XXX-short-description
```

**Examples:**
- `task/003-context-menus`
- `task/004-add-version`
- `task/007-delete-operations`

**For non-task work:**
- `fix/bug-description` - Bug fixes
- `docs/documentation-update` - Documentation only
- `refactor/component-name` - Code refactoring
- `test/feature-name` - Test additions

### Workflow for Every Task

**1. Before Starting:**
```bash
git checkout main
git pull origin main
git checkout -b task/XXX-short-name
```

**Claude MUST ask:**
> "Starting Task XXX. First, let's create a feature branch:
> - Branch name: `task/XXX-short-description`
> - Shall I provide the git commands?"

**2. During Work:**
```bash
# Commit frequently
git add [files]
git commit -m "feat(XXX): implement [feature]"
git push -u origin task/XXX-short-name
```

**3. Before Task Completion - Run Tests:**
```bash
npm run test           # Unit tests
npm run lint          # Linting
npm run compile       # TypeScript compilation
```

**Claude MUST enforce:**
> "Before we can complete this task, we need to verify:
> 1. All tests passing
> 2. No linting errors
> 3. TypeScript compiles successfully
>
> Shall I help you run these checks?"

**4. Merge to Main (only when tests pass):**
```bash
git checkout main
git pull origin main
git merge task/XXX-short-name
npm run test && npm run compile  # Verify after merge
git push origin main
git branch -d task/XXX-short-name
```

### Git Best Practices

**DO:**
- ✅ Create branch before starting work
- ✅ Commit frequently with clear messages
- ✅ Run tests before merging
- ✅ Keep branches short-lived (1-2 days max)
- ✅ Delete branches after merging

**DON'T:**
- ❌ Commit directly to main
- ❌ Merge without running tests
- ❌ Push broken code to main
- ❌ Leave branches unmerged for weeks
- ❌ Mix multiple tasks in one branch

## Test-Driven Development (TDD) - MANDATORY

### TDD Cycle - ALWAYS Follow

```
1. RED    → Write a failing test
2. GREEN  → Write minimal code to pass
3. REFACTOR → Improve code while keeping tests green
```

**CRITICAL: Write tests BEFORE implementation code.**

### TDD Workflow

**Phase 1: RED - Write Failing Test**

```typescript
// Example: test/commands/copyCommands.test.ts
describe('copyGroupIdCommand', () => {
    it('should copy group ID to clipboard', async () => {
        const mockNode = { groupId: 'test-group' };
        await copyGroupIdCommand(mockNode);
        expect(mockClipboard.writeText).toHaveBeenCalledWith('test-group');
    });
});
```

Run test - it MUST fail:
```bash
npm run test  # Expected: FAIL ❌

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Apicurio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
