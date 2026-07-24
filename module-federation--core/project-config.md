---
trigger: always_on
description: - **ALWAYS PARALLELIZE**: Break down ANY task into multiple parallel sub-tasks
---

# Claude Task Parallelization & Development Guidelines

## Core Principles

### Maximum Parallelization Strategy
- **ALWAYS PARALLELIZE**: Break down ANY task into multiple parallel sub-tasks
- **IMMEDIATE EXECUTION**: Launch parallel Task agents immediately without asking questions
- **NO CLARIFICATION**: Skip asking about implementation details unless absolutely critical
- **CONTEXT EFFICIENCY**: Each task handles only its specific scope to minimize token usage

## Universal Parallel Task Patterns

### For Any Codebase Task
Analyze the request and immediately split into parallel tasks based on:

1. **File Type Separation**
   - Source code files
   - Test files
   - Configuration files
   - Documentation files
   - Build/deployment files

2. **Functional Separation**
   - Core implementation
   - Helper/utility functions
   - Integration points
   - Data models/schemas
   - External interfaces

3. **Directory-Based Separation**
   - Different modules/packages
   - Frontend vs backend
   - Library vs application code
   - Public vs private APIs

### Dynamic Task Allocation Examples

#### Example 1: Feature Implementation (5-7 parallel tasks)
- Task 1: Core feature logic
- Task 2: Supporting utilities/helpers
- Task 3: Tests (if applicable)
- Task 4: Integration with existing code
- Task 5: Configuration updates
- Task 6: Type definitions (if typed language)
- Task 7: Remaining changes and coordination

#### Example 2: Bug Investigation (3-10 parallel tasks)
- Task 1: Search for error patterns in logs/code
- Task 2: Analyze related test files
- Task 3: Check recent commits/changes
- Task 4: Investigate similar issues in codebase
- Task 5: Review documentation/comments

#### Example 3: Refactoring (4-6 parallel tasks)
- Task 1: Identify all usage locations
- Task 2: Plan new structure
- Task 3: Update core implementations
- Task 4: Update dependent code
- Task 5: Update tests
- Task 6: Verify no breaking changes

#### Example 4: Codebase Analysis (5-8 parallel tasks)
- Task 1: Analyze file structure
- Task 2: Review core business logic
- Task 3: Map dependencies
- Task 4: Identify patterns/conventions
- Task 5: Review configuration
- Task 6: Analyze build system
- Task 7: Check documentation
- Task 8: Security/performance patterns

### Adaptive Parallelization Rules
1. **Minimum 3 tasks**: Even simple requests should use at least 3 parallel tasks
2. **Maximum 10 tasks**: Avoid over-fragmentation; combine related work
3. **File count based**: 
   - 1-5 files: 3-4 tasks
   - 6-20 files: 5-7 tasks
   - 20+ files: 7-10 tasks
4. **Complexity based**:
   - Simple changes: 3-4 tasks
   - Medium complexity: 5-7 tasks
   - High complexity: 7-10 tasks

### Post-Execution Coordination
Always include a final review task that:
- Synthesizes findings from all parallel tasks
- Resolves any conflicts or overlaps
- Runs verification commands (lint, test, build)
- Provides unified summary of changes

## Context Optimization Strategies

### Code Reading Rules
- **STRIP COMMENTS**: Remove all comments when analyzing existing code
- **FOCUSED SCOPE**: Each task reads ONLY files relevant to its specific responsibility
- **MINIMIZE CONTEXT**: Avoid loading unnecessary files to preserve token efficiency

### File Management
- **PREFER MODIFICATION**: Always edit existing files over creating new ones
- **MINIMAL CHANGES**: Make the smallest possible changes to achieve functionality
- **PRESERVE PATTERNS**: Maintain existing code style and architectural patterns

## Implementation Guidelines

### Critical Rules
1. **PRESERVE ARCHITECTURE**: Never change existing patterns without explicit request
2. **FOLLOW CONVENTIONS**: Match existing naming, file organization, and code style
3. **REUSE COMPONENTS**: Use existing utilities, hooks, and components before creating new ones
4. **ATOMIC CHANGES**: Each task makes self-contained, non-conflicting modifications
5. **TEST COVERAGE**: Maintain or improve test coverage with changes
6. **BACKWARDS COMPATIBILITY**: Ensure changes don't break existing functionality
7. **PERFORMANCE AWARE**: Consider performance implications of changes
8. **SECURITY CONSCIOUS**: Never introduce security vulnerabilities

### Efficiency Practices
- Launch all tasks in a single message using multiple tool invocations
- Each task should have clear, non-overlapping responsibilities
- Consolidate small related changes into Task 7 to prevent over-fragmentation

### Error Handling
- Each task should handle its own errors gracefully
- If a task encounters blockers, it should document them clearly
- The review task should identify and resolve integration issues
- Always provide actionable error messages
- Suggest fixes for common errors encountered
- Log errors with context for debugging
- Fail fast but provide recovery options

## Parallelization Examples

### Example: "Fix the login bug"
```
Claude: [Launches 5 parallel tasks immediately]
- Task 1: Search for login-related code and error patterns
- Task 2: Check recent commits touching authentication
- Task 3: Analyze login tests and error logs
- Task 4: Review auth configuration and dependencies
- Task 5: Investigate similar issues and edge cases
```

### Example: "Add a new API endpoint"
```
Claude: [Launches 6 parallel tasks immediately]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [module-federation/core](https://github.com/module-federation/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
