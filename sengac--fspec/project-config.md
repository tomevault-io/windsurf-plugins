---
trigger: always_on
description: This document defines the complete workflow for managing work (project management) and specifications (Gherkin features) when building this project.
---

# Project Management and Specification Guidelines for fspec

This document defines the complete workflow for managing work (project management) and specifications (Gherkin features) when building this project.

## CRITICAL: Project Management FIRST, Specifications SECOND

**Before writing any Gherkin specifications or code, you MUST manage work using fspec's project management system.**

## Project Management Workflow (STEP 1)

### Understanding Work Organization

fspec uses a Kanban-based project management system with:

- **Work Units**: Discrete pieces of work (e.g., AUTH-001, DASH-002)
- **Prefixes**: Short codes namespacing work unit IDs (AUTH, DASH, API, SEC, PERF)
- **Epics**: High-level business initiatives containing multiple work units
- **Kanban States**: backlog → specifying → testing → implementing → validating → done (+ blocked)

### Before Starting ANY Work

1. **Check what needs to be done**: `fspec list-work-units --status=backlog`
2. **Pick a work unit**: Review the backlog and choose the next highest priority item
3. **Move to specifying**: `fspec update-work-unit-status WORK-001 specifying`
4. **Now proceed to write specifications** (see Specification Workflow below)

### Managing Your Work Units

```bash
# List all work units
fspec list-work-units

# Show details of a specific work unit
fspec show-work-unit WORK-001

# Create a new work unit (if planning new work)
fspec create-story PREFIX "Title" --description "Details" --epic=epic-name  # For features
fspec create-bug PREFIX "Title" --description "Details" --epic=epic-name    # For bug fixes
fspec create-task PREFIX "Title" --description "Details" --epic=epic-name   # For tasks

# Set user story fields for work unit (used during Example Mapping)
fspec set-user-story WORK-001 --role "user role" --action "what they want" --benefit "why they want it"

# Move work unit through Kanban workflow
fspec update-work-unit-status WORK-001 specifying   # Writing specs
fspec update-work-unit-status WORK-001 testing      # Writing tests
fspec update-work-unit-status WORK-001 implementing # Writing code
fspec update-work-unit-status WORK-001 validating   # Code review/testing
fspec update-work-unit-status WORK-001 done         # Completed

# Mark work unit as blocked (with reason)
fspec update-work-unit-status WORK-001 blocked --blocked-reason "Waiting for external API documentation"
```

### ACDD with Project Management

**Acceptance Criteria Driven Development (ACDD)** combined with project management:

1. **Pick work unit** from backlog → move to `specifying`
2. **Write specifications** (Gherkin feature files) → move to `testing`
3. **Write tests** that map to scenarios → move to `implementing`
4. **Write code** to make tests pass → move to `validating`
5. **Review/validate** code and specs → move to `done`

### Moving Backward Through Kanban States

**CRITICAL**: You CAN and SHOULD move work units backward when mistakes are discovered, rather than creating new work units.

**When to Move Backward:**

- **From testing → specifying**: Tests revealed incomplete or wrong acceptance criteria
- **From implementing → testing**: Need to add or fix test cases
- **From implementing → specifying**: Discovered missing scenarios or acceptance criteria
- **From validating → implementing**: Quality checks failed, need more implementation
- **From validating → testing**: Tests are inadequate or need refactoring
- **From any state → specifying**: Fundamental misunderstanding of requirements

**How to Move Backward:**

```bash
# Example: Realized specifications are incomplete while writing tests
fspec update-work-unit-status AUTH-001 specifying

# Example: Quality checks failed during validation, need to fix code
fspec update-work-unit-status AUTH-001 implementing

# Example: Need to refactor tests based on implementation learnings
fspec update-work-unit-status AUTH-001 testing
```

**Why Move Backward (Not Create New Work Units):**

✅ **DO** move backward when:
- You discover incomplete specifications
- Tests don't adequately cover scenarios
- Implementation revealed gaps in acceptance criteria
- Quality checks uncovered issues requiring earlier phase work
- You realize you misunderstood requirements

❌ **DON'T** create new work units for:
- Fixing mistakes in current work unit
- Refining existing specifications
- Improving existing tests
- Correcting implementation errors

**When to Create New Work Units:**

Create new work units only for:
- **Genuinely new features** not part of current work
- **Out of scope** enhancements discovered during work
- **Technical debt** or refactoring that should be tracked separately
- **Bugs** discovered in already-completed work units (marked `done`)

**Example Workflow with Backward Movement:**

```bash
# 1. Start work
fspec update-work-unit-status AUTH-001 specifying
# ... write specifications

# 2. Move to testing
fspec update-work-unit-status AUTH-001 testing
# ... start writing tests

# 3. DISCOVER: Specs are incomplete!
# Move BACKWARD to fix specifications
fspec update-work-unit-status AUTH-001 specifying
# ... add missing scenarios

# 4. Specifications complete, return to testing
fspec update-work-unit-status AUTH-001 testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sengac/fspec](https://github.com/sengac/fspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
