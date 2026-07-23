---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT: Object-Oriented Architecture

**This is NOT a typical functional JavaScript project!**

Lively4 follows **object-oriented principles** with class hierarchies and inheritance:

- **Use inheritance properly**: If functionality needs to be shared, put it in the base class
- **Single source of truth**: Don't duplicate methods across subclasses - use the parent class
- **Polymorphism**: Subclasses inherit and can override parent methods when needed
- **Class hierarchy matters**: `LivelyChat` → `OpenaiRealtimeChat`, `LivelyOpencode`, `LivelyAiWorkspace`

**Example: Database Write Guards**
- ❌ WRONG: Add `canWriteToDatabase()` to each component separately
- ✅ RIGHT: Add `canWriteToDatabase()` to `LivelyChat` base class, all subclasses inherit it

**Example: Composition Pattern - Propagating State**

When a parent component contains child components, state changes must be explicitly propagated:

```javascript
// ❌ WRONG: Only set state on parent
enableReplay() {
  this._replayMode = true;  // Only affects parent, children still write to DB!
}

// ✅ RIGHT: Propagate state to composed children
enableReplay() {
  this._replayMode = true;

  // CRITICAL: Propagate to child components
  this.realtimeComponent._replayMode = true;
  this.opencodeComponent._replayMode = true;
}

disableReplay() {
  this._replayMode = false;

  // CRITICAL: Clear from child components
  if (this.realtimeComponent) this.realtimeComponent._replayMode = false;
  if (this.opencodeComponent) this.opencodeComponent._replayMode = false;
}
```

**Key principle**: Child components don't automatically inherit instance variables from their container. State must be explicitly synchronized in composition relationships.

When you find yourself duplicating code across components that share a base class, **STOP** and move it to the parent class instead.

## AI Collaboration Experiment

A significant part of this AI collaboration is an **experiment to teach Claude Code how to develop in Lively4**. By working together on real development tasks, we are:

- Teaching Claude the patterns, conventions, and workflows of the Lively4 system
- Documenting these learnings for both AI and human developers
- Creating examples and best practices through hands-on exploration
- Building up Claude's understanding of the self-supporting development environment

This documentation serves dual purposes: guiding AI development work and creating human-readable documentation of Lively4's development practices. The `demos/claude/` directory contains examples and experiments from this collaborative learning process.

## CRITICAL: Git Safety Guidelines

**How to safely work with git and preserve all user work!**

**Guiding Principles:**
- ✅ **ALWAYS** wait for explicit user request before running any git commands
- ✅ **ALWAYS** check `git status` and `git diff` first to see the current state
- ✅ **ALWAYS** ask user for confirmation before making destructive changes
- ✅ **ALWAYS** verify what other uncommitted work exists before any operation

**Example of WRONG approach:**
```javascript
// User: "Let's scrap this feature"
// ❌ WRONG - immediately destroying work:
bash("git checkout -- src/file.js")

// What if user had OTHER uncommitted changes?
// What if the file had important work-in-progress?
// DISASTER!
```

**Example of CORRECT approach:**
```javascript
// User: "Let's scrap this feature"

// Step 1: Check current state
bash("git status")
// Shows: "modified: src/file.js, modified: src/other-important-file.js"

// Step 2: Check what will be lost
bash("git diff src/file.js")
// Shows the actual changes

// Step 3: Ask user for confirmation
"I see you have changes in src/file.js. You also have uncommitted changes in 
src/other-important-file.js. Should I discard the changes in src/file.js, 
or do you want to keep working on it?"

// Step 4: Only proceed if user explicitly confirms
// Then run the git command they requested
```

**Why this approach is safe:**
- Users may have hours of uncommitted work
- Git operations can permanently destroy uncommitted changes
- User gets to review what will be lost before deciding
- Verifies no other important work will be affected

## Essential Commands

**Testing:**

Use the MCP testing tools for in-browser test execution with minimal token usage:

**Phase 1: Run Tests**
```javascript
// Run a single test file
mcp__lively4__run-tests(testPath: "test/client/strings-test.js")

// Run ALL tests (minimal output)
mcp__lively4__run-tests(runAll: true)
// Returns: "✅ All green! 456 tests passed across 77 files"
// or: "❌ 18 tests failed (438 passed) across 5 files: ..."

// Filter tests with grep pattern
mcp__lively4__run-tests(testPath: "test/client/strings-test.js", grep: "toUpperCaseFirst")

// Errors-only mode for single files
mcp__lively4__run-tests(testPath: "test/client/strings-test.js", errorsOnly: true)
```

**Phase 2: Inspect Results (Hierarchical Navigation)**

Supports 3 levels of detail - navigate like a directory tree:

```javascript
// Level 1: Summary view (like `ls`) - shows file-level counts only
mcp__lively4__inspect-test-results()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LivelyKernel/lively4-core](https://github.com/LivelyKernel/lively4-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
