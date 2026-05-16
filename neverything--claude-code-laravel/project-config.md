---
trigger: always_on
description: We're building production-quality Laravel applications together. Your role is to create maintainable, efficient solutions while following Laravel best practices and catching potential issues early.
---

# Laravel Development Partnership

We're building production-quality Laravel applications together. Your role is to create maintainable, efficient solutions while following Laravel best practices and catching potential issues early.

When you seem stuck or overly complex, I'll redirect you - my guidance helps you stay on track.

## 🚨 AUTOMATED CHECKS ARE MANDATORY
**ALL hook issues are BLOCKING - EVERYTHING must be ✅ GREEN!**  
No errors. No formatting issues. No linting problems. Zero tolerance.  
These are not suggestions. Fix ALL issues before continuing.

## CRITICAL WORKFLOW - ALWAYS FOLLOW THIS!

### Research → Plan → Implement
**NEVER JUMP STRAIGHT TO CODING!** Always follow this sequence:
1. **Research**: Explore the codebase, understand existing patterns
2. **Plan**: Create a detailed implementation plan and verify it with me  
3. **Implement**: Execute the plan with validation checkpoints

When asked to implement any feature, you'll first say: "Let me research the codebase and create a plan before implementing."

For complex architectural decisions or challenging problems, use **"ultrathink"** to engage maximum reasoning capacity. Say: "Let me ultrathink about this architecture before proposing a solution."

### USE MULTIPLE AGENTS!
*Leverage subagents aggressively* for better results:

* Spawn agents to explore different parts of the codebase in parallel
* Use one agent to write tests while another implements features
* Delegate research tasks: "I'll have an agent investigate the database schema while I analyze the API structure"
* For complex refactors: One agent identifies changes, another implements them

Say: "I'll spawn agents to tackle different aspects of this problem" whenever a task has multiple independent parts.

### Reality Checkpoints
**Stop and validate** at these moments:
- After implementing a complete feature
- Before starting a new major component  
- When something feels wrong
- Before declaring "done"
- **WHEN HOOKS FAIL WITH ERRORS** ❌

Run: `composer refactor && composer test && composer lint`

> Why: You can lose track of what's actually working. These checkpoints prevent cascading failures.

### 🚨 CRITICAL: Hook Failures Are BLOCKING
**When hooks report ANY issues (exit code 2), you MUST:**
1. **STOP IMMEDIATELY** - Do not continue with other tasks
2. **FIX ALL ISSUES** - Address every ❌ issue until everything is ✅ GREEN
3. **VERIFY THE FIX** - Re-run the failed command to confirm it's fixed
4. **CONTINUE ORIGINAL TASK** - Return to what you were doing before the interrupt
5. **NEVER IGNORE** - There are NO warnings, only requirements

This includes:
- Formatting issues (Pint)
- Linting violations (PHPStan/Larastan)
- Code quality issues (Rector)
- Test failures (Pest)
- ALL other checks

Your code must be 100% clean. No exceptions.

**Recovery Protocol:**
- When interrupted by a hook failure, maintain awareness of your original task
- After fixing all issues and verifying the fix, continue where you left off
- Use the todo list to track both the fix and your original task

## Working Memory Management

### When context gets long:
- Re-read this CLAUDE.md file
- Summarize progress in a PROGRESS.md file
- Document current state before major changes

### Maintain TODO.md:
```
## Current Task
- [ ] What we're doing RIGHT NOW

## Completed  
- [x] What's actually done and tested

## Next Steps
- [ ] What comes next
```

## Laravel/PHP-Specific Rules

### FORBIDDEN - NEVER DO THESE:
- **NO raw SQL** - use Eloquent or Query Builder
- **NO direct $_GET/$_POST** - use Laravel's request validation
- **NO keeping old and new code together**
- **NO** migration functions or compatibility layers
- **NO** versioned function names (processV2, handleNew)
- **NO** custom exception hierarchies without good reason
- **NO** TODOs in final code
- **NO** inline styles or scripts in Blade templates - use Tailwind classes
- **NO** database queries in Livewire render methods
- **NO** polling in Livewire/Filament - use Laravel Reverb + Echo for real-time updates
- **NO** comments within methods - let code be self-documenting with clear variable names
- **NO** class constants for labels/colors/icons - use Enum classes with methods

> **AUTOMATED ENFORCEMENT**: The smart-lint hook will BLOCK commits that violate these rules.  
> When you see `❌ FORBIDDEN PATTERN`, you MUST fix it immediately!

### Required Standards:
- **Delete** old code when replacing it
- **Meaningful names**: `$userId` not `$id`, `$userAccountBalance` not `$balance`
- **Early returns** to reduce nesting
- **Type hints** on all methods: `public function handle(User $user): bool`
- **Self-documenting code** with clear variable names - no inline comments
- **Enum classes** with methods like `getLabel()`, `getDescription()`, `getColor()`, `getIcon()` (Filament style)
- **Pest tests** for all business logic, never PHPUnit always Pest tests
- **Livewire actions** for user interactions, not direct method calls
- **Filament resources** follow the standard pattern
- **Eloquent relationships** properly defined with return types
- **Laravel Echo** for real-time updates instead of polling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neverything/claude-code-laravel](https://github.com/neverything/claude-code-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
