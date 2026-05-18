---
trigger: always_on
description: Knowledge and patterns for creating effective Claude Code skills.
---

# Agent Skill Design Guide

Knowledge and patterns for creating effective Claude Code skills.

## Core Principles

### 1. Single Responsibility
Each skill should have **one clear purpose**. Don't combine fundamentally different workflows into a single skill with flags.

**Good:** Separate skills for local-only vs. remote-push workflows
**Bad:** One skill with a `--remote` flag that changes core behavior

**Why:** Separate skills make intent explicit, reduce accidental misuse, and are easier to understand.

### 2. Name Consistency
The skill name in YAML frontmatter **must match** the directory name exactly.

```markdown
---
name: implement-with-feedback
description: ...
---
```
Directory: `implement-with-feedback/SKILL.md`

### 3. When to Offer
The `description` field should help Claude know **when to proactively offer** the skill.

**Good patterns:**
- "Use when starting any plan-based implementation task"
- "Use when the user asks to build web components, pages, or applications"
- "Analyze other agents' sessions and construct targeted corrective prompts"

**Include triggering scenarios** so Claude can pattern-match user requests.

### 4. Argument Hints
Use `argument-hint` to show expected parameters:

```yaml
argument-hint: <branch-type>/<description> [plan-file]
```

This helps both the user and Claude understand how to invoke the skill.

## Workflow Structure

### Phase-Based Organization
Break complex workflows into clear, numbered phases:

```markdown
### Phase 1: Pre-flight Checks
### Phase 2: Branch Creation
### Phase 3: WIP Progress File
### Phase 4: Implementation Loop
### Phase 5: Completion
```

**Benefits:**
- Easy to follow
- Clear checkpoints
- Natural place for validation steps
- Resume-friendly (can pick up at any phase)

### Pre-flight Checks
Always start with validation:
- Check working tree state
- Verify branch
- Confirm prerequisites exist
- Pull latest changes

**Pattern:**
```markdown
1. **Verify X.** Run `command`. If condition fails, **STOP** and tell the user:
   > "Error message here."
   Do NOT proceed until fixed.
```

The explicit "STOP" and "Do NOT proceed" language is critical for Claude to respect boundaries.

## Rules Section

### Stating Constraints
Use **NEVER** for hard boundaries:

```markdown
## Rules

- **NEVER force push.** History is sacred in this workflow.
- **NEVER amend pushed commits.** Make a new commit instead.
- **NEVER skip pushing.** Every commit must be pushed immediately.
```

**Why NEVER works:**
- Unambiguous
- Easy to search for
- Creates strong behavioral constraints
- No room for interpretation

### Commit Discipline
If the skill involves git, specify commit patterns:

```markdown
- `feat: add auth middleware for API routes`
- `fix: handle null response from scanner`
- `wip: partial implementation of results table`
- `docs: update scanner authoring guide`
- `test: add normalizer tests for ffuf`
- `refactor: extract fingerprint logic to shared util`
```

Provide examples for each commit type the agent might need.

## Living Documentation Pattern

Skills can create **progress tracking files** that evolve during execution:

```markdown
### Phase 3: WIP Progress File

1. **Create `docs/plans/plan_<branch-name>_implementation.md`**
2. **Initial content:** [template here]
3. **Commit the WIP file immediately**
```

**Benefits:**
- Self-documenting work
- Useful for monitoring
- Creates reviewable history
- Helps agent track progress
- Provides context for future sessions

**Key elements of WIP files:**
- Status field
- Task checklist
- Progress log with timestamps
- Decisions & notes section
- Blockers section
- Commit log

## Common Patterns

### Argument Handling
```markdown
If `$ARGUMENTS` is provided, use it as the branch name directly
(e.g. `/implement-with-feedback feature/add-auth`).
Otherwise, ask the user what kind of work this is and derive a branch name.
```

Make it work with or without arguments.

### Conditional Behavior
```markdown
**Verify we are on main/master.** If not, warn the user and ask whether
to continue from the current branch or switch to main first.
```

Don't assume - ask when there's ambiguity.

### Completion Actions
```markdown
3. **Inform the user** the branch is ready for review / PR creation.
   Offer to create the PR.
```

Always offer next logical steps at completion.

## When to Split Skills

Consider separate skills when:

1. **Core behavior differs** (local vs. remote, sync vs. async)
2. **Safety requirements differ** (destructive vs. safe operations)
3. **Target audience differs** (solo vs. team, internal vs. external)
4. **Workflow branches significantly** (not just parameter tweaks)

Consider combining when:
1. Only presentation differs
2. It's truly just an optional parameter
3. The core flow is identical
4. Safety constraints are the same

## Skill Metadata

### Required Fields
```yaml
---
name: skill-name
description: Clear description with "Use when..." triggers
---
```

### Optional Fields
```yaml
argument-hint: <type>/<name> [optional-param]
```

## Testing & Iteration

### Validation Checklist
- [ ] Name matches directory
- [ ] Description includes "Use when..." triggers
- [ ] Pre-flight checks prevent bad states
- [ ] Rules section uses NEVER for boundaries
- [ ] Phases are clearly numbered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xalior/agent-skills](https://github.com/Xalior/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
