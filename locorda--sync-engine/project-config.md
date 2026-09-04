---
trigger: always_on
description: **Locorda** is a Dart/Flutter library enabling offline-first applications that sync seamlessly to passive storage backends (Solid Pods, Google Drive, etc.) using state-based CRDTs for conflict-free collaboration. This is a **monorepo** using **Melos** for multipackage management.
---

# Locorda: AI Coding Agent Instructions

## Project Overview

**Locorda** is a Dart/Flutter library enabling offline-first applications that sync seamlessly to passive storage backends (Solid Pods, Google Drive, etc.) using state-based CRDTs for conflict-free collaboration. This is a **monorepo** using **Melos** for multipackage management.

**⚠️ Critical**: Specification in `spec/` is outdated. Implementation has diverged significantly. **Always prioritize actual code over spec documentation**.

## Agent Documentation & Lifecycle Rules

### 1. Storage & Organization
If you need it, you can generate Markdown documentation for your plans, summaries, or test reports. Follow these rules (they do not apply if I ask you explicitly to generate a markdown file for a specific purpose):
- **Primary Folder:** All agent-generated documentation must live in `.agents/`.
- **Daily Folders:** Use sub-folders formatted as `YYYY-MM-DD` (e.g., `.agents/2026-02-11/`).
- **Archive Sub-folders:** When a task or phase is completed, or if a plan is superseded by manual human code changes, move the relevant files into `.agents/YYYY-MM-DD/archive/`.

### 2. Guardrails Against "Bad Context"
- **Code is the Ground Truth:** The current state of the code in `lib/` or `src/` always overrides any notes in `.agents/`. 
- **Stale Context:** Do not follow plans in the `archive/` folder unless explicitly asked to review history. 
- **Conflict Resolution:** If you detect that the user has manually changed code in a way that contradicts a previous `.md` plan, immediately move that plan to `archive/` and generate a brief `SUPERSEDED.md` note explaining the new direction.

### 3. File Hygiene
- Use descriptive prefixes: `PLAN_`, `SUMMARY_`, `TEST_`.
- Do not clutter the root or package directories with Markdown reports.
- If the project is in a "stable" state, you may suggest a "History Compression" where you merge multiple old reports into a single `ARCHIVE_SUMMARY.md` for that date.

## 🛑 MANDATORY: Ask Before Code Edits (With Exceptions)

**CRITICAL WORKFLOW RULE**

Before calling `replace_string_in_file`, `create_file`, or any edit tool, you MUST:

1. ✅ Analyze the problem and develop a solution
2. ✅ Explain the proposed fix (show code example if helpful)
3. ⚠️ **EXPLICITLY ASK: "Shall I implement this change?"**
4. ⏸️ **WAIT for explicit approval** (user says: "yes"/"approve"/"do it")
5. ✅ Only THEN edit the file

**Exception**: When the user gives a **direct, specific instruction** for exactly what to do:
- ✅ "Translate this document to English" → Just do it
- ✅ "Rename variable X to Y" → Just do it
- ✅ "Add parameter Z to function F" → Just do it
- ❌ "The code has a bug" → Ask first (solution unclear)
- ❌ "Improve performance" → Ask first (approach unclear)

**This applies to everything else:**
- ✗ Bug fixes where solution is not explicitly specified → Ask first
- ✗ Refactoring/optimization without clear direction → Ask first
- ✗ New features without specific implementation → Ask first
- ✗ Test changes where approach unclear → Ask first
- ✗ "Obvious" fixes where action not specified → Ask first

**No rationalizations allowed:**
- "This is just a small fix" → Still ask (unless explicitly instructed)
- "The user will obviously want this" → Still ask
- "I already explained the solution" ≠ "I have permission to implement"

**Checklist before editing:**
```
□ Is this a direct instruction with clear action?
  → YES: Proceed
  → NO: Continue checklist
□ Have I explicitly asked: "Shall I implement this change?"
□ Has the user replied: "yes"/"approve"/"do it"?
□ If NO to either → STOP and ask first
```

## Architecture: 4-Layer Design

1. **Data Resource Layer**: Clean RDF using standard vocabularies (schema.org)
2. **Merge Contract Layer**: Property-level CRDT rules via `sync:` and `algo:` vocabularies
3. **Indexing Layer**: Performance via sharded indices (`idx:` vocab) - supports FullIndex (monolithic) and GroupIndex (partitioned)
4. **Sync Strategy Layer**: App-controlled sync patterns with RootResourceFetchPolicy (onRequest/prefetch)

**Key Innovation**: Hybrid Logical Clocks combine causality tracking (logical time) with intuitive tie-breaking (physical timestamps).

## Package Structure

```
packages/
├── locorda           # Main entry point, docs, examples
├── locorda_core      # Platform-agnostic CRDT sync engine (pure Dart)
├── locorda_annotations # CRDT merge strategy annotations  
├── locorda_drift     # Drift (SQLite) storage backend
├── locorda_solid     # Solid Pod integration utilities
├── locorda_solid_auth # Solid authentication (Flutter + solid-auth)
└── locorda_solid_ui  # Flutter UI components (login, sync status)
```

**Dependency Rule**: No circular deps, no re-exports between packages, clean separation.

## Essential Commands

### Setup & Testing
```bash
# Initial setup after clone
dart pub run melos bootstrap

# Run tests with coverage (PREFERRED)
dart tool/run_tests.dart

# All packages test
dart pub run melos test

# Record mode for test expectations (overwrites files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locorda/sync-engine](https://github.com/locorda/sync-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
