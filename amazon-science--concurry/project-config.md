---
trigger: always_on
description: **NEVER create standalone summary `.md` files** documenting implementation changes, fixes, refactorings, or feature additions. These waste tokens and are never read.
---

# Documentation Practices: NO SUMMARY DOCUMENTS

## ⚠️ CRITICAL RULE: NEVER CREATE SUMMARY DOCUMENTS

**NEVER create standalone summary `.md` files** documenting implementation changes, fixes, refactorings, or feature additions. These waste tokens and are never read.

❌ **DO NOT CREATE FILES LIKE**:
- `FIX_SUMMARY.md`
- `IMPLEMENTATION_SUMMARY.md`
- `REFACTORING_COMPLETE.md`
- `FEATURE_IMPLEMENTATION.md`
- `CHANGES_SUMMARY.md`
- Any other standalone documentation of completed work

## Instead: Four-Part Documentation Strategy

All information from implementation sessions must flow into these four locations:

### 1. **Chat Summary** (Immediate Communication)

At the end of each implementation session, provide a **structured summary in the chat** with these sections:

```markdown
## Implementation Summary

### What Was Changed
- Brief list of files modified and their changes
- High-level description of the fix/feature

### Why These Changes Were Needed
- Root cause of the issue (if a fix)
- Rationale for the design decision (if a feature)
- Problems with previous implementation (if a refactoring)

### Alternatives Considered
- What other approaches were evaluated
- Why they were rejected
- Trade-offs of the chosen approach

### Testing
- New testcases added (with file paths)
- Existing testcases updated (with file paths)
- Edge cases now covered

### Documentation Updates
- Architecture docs updated (with file paths)
- User guide sections updated (with file paths)
- API docstrings updated (with file/class/method references)

### Other Changes
- Configuration changes
- Dependencies added/removed
- Breaking changes (if any)
```

**This chat summary is the ONLY place for quick, session-level documentation.**

---

### 2. **Architecture Documentation** (Historical + Technical)

Location: `concurry/docs/architecture/`

**When to Update**: For ANY architectural change, design decision, or significant refactoring.

**What to Include**:
1. **Current Implementation**: How the system works now
2. **Historical Context**: Previous implementations and their issues
3. **Why the Change**: What problems were encountered
4. **Example Code**: For each approach (previous and current)
5. **Trade-offs**: Pros and cons of the design

**Format Example**:

```markdown
## Feature: Worker Submission Queue

### Current Implementation (Version 3 - October 2025)

[Detailed explanation of current approach]

**Example:**
```python
# Current implementation code
```

**Why This Works:**
- Reason 1
- Reason 2

---

### Previous Implementation (Version 2 - September 2025)

**Approach:**
[Description of previous approach]

**Example:**
```python
# Previous implementation code
```

**Issues Encountered:**
1. Race condition when stop() called during submission
2. Semaphore not released if worker stopped
3. Futures could be returned after pool stopped

**Why It Failed:**
[Detailed explanation]

---

### Initial Implementation (Version 1 - August 2025)

[Similar structure]
```

**Key Files**:
- `docs/architecture/workers.md` - Worker system architecture
- `docs/architecture/synchronization.md` - wait()/gather() and futures
- `docs/architecture/limits.md` - Rate limiting and resource limits
- `docs/architecture/retries.md` - Retry logic and configuration
- `docs/architecture/configuration.md` - Global config system
- `docs/architecture/task-worker.md` - TaskWorker specifics

**Create NEW architecture docs** when implementing completely new subsystems.

---

### 3. **User Guide + API Docstrings** (User-Facing Behavior)

Location: `concurry/docs/user-guide/` and module docstrings

**When to Update**: ONLY when user-facing behavior changes.

**DO Update When**:
- ✅ New public API added
- ✅ Public API signature changed
- ✅ New parameters or options available
- ✅ Default behavior changed
- ✅ New execution mode added
- ✅ New feature users can interact with

**DO NOT Update When**:
- ❌ Backend implementation fixed (no behavior change)
- ❌ Internal refactoring (same external API)
- ❌ Performance optimization (no API change)
- ❌ Bug fix that restores expected behavior

**User Guide Structure**:
```markdown
## Feature Name

### Overview
Brief description of what the feature does and when to use it.

### Basic Usage
```python
# Minimal working example
```

### Common Patterns
```python
# Pattern 1: Common use case
```

```python
# Pattern 2: Another common use case
```

### Advanced Usage
```python
# Complex example with all options
```

### Configuration
- List of relevant config keys (reference, not full docs)
- Link to API docs for details

### Troubleshooting
Common issues and solutions.
```

**API Docstrings**:
- Include parameter descriptions
- Reference config keys for defaults (e.g., "Defaults to `global_config.defaults.worker_timeout`")
- Provide usage examples
- Document exceptions raised
- **NEVER hardcode default values in docstrings**

**Global Config Defaults**:
- ❌ DO NOT document all config keys in user guide
- ✅ DO mention relevant config keys in context
- ✅ Full config documentation belongs in API reference

---

### 4. **Testcases** (Executable Edge Case Documentation)

Location: `concurry/tests/` (in appropriate subdirectories)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
