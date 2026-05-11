---
trigger: always_on
description: > **This is a living document.** Agents working on this codebase are expected to improve it.
---

# Claudio Development Guidelines

> **This is a living document.** Agents working on this codebase are expected to improve it.
> See [Self-Improvement Protocol](#self-improvement-protocol) for details.

## Self-Improvement Protocol

### Mandate

When you complete a task on this codebase, review this file (and any relevant directory-level `AGENTS.md`) and make meaningful updates based on what you learned. This is not optional — it is part of completing your work. Updates should be included in your commit alongside the code changes.

### What Constitutes a Meaningful Update

**DO update when you discover:**
- A codebase pattern or convention not already documented here (add to [Codebase Patterns](#codebase-patterns))
- A pitfall, gotcha, or subtle bug you hit or nearly hit (add to [Known Pitfalls](#known-pitfalls))
- A new package, significant type, or architectural component (update [Architecture Map](#architecture-map))
- A testing trick or technique specific to this codebase (add to [Testing Notes](#testing-notes))
- A correction to something already written here that turned out to be wrong (fix it directly)
- A dependency quirk or build/toolchain issue (add to [Build & Toolchain](#build--toolchain))

**DO NOT update for:**
- Generic Go knowledge that any Go developer would know
- Restating what's already documented in this file
- Trivial observations that won't help future agents
- Speculative advice not grounded in actual experience on this codebase

### How to Update

1. Add entries to the appropriate section below
2. Keep entries concise — one to two sentences, with a code reference where relevant
3. If a section grows beyond ~15 entries, reorganize or split it
4. If you discover something here is wrong, fix or remove it — don't leave stale knowledge
5. Preserve the structure and heading hierarchy of this document

### Directory-Scoped Guidelines

When your knowledge is specific to a single package or directory, put it in that directory's `AGENTS.md` instead of here. If one already exists, update it. If not, create it. These are living documents just like this root file — the self-improvement mandate applies to all of them.

**When creating a new directory-level `AGENTS.md`:**
1. Create `AGENTS.md` in the target directory with package-specific guidance
2. Create a `CLAUDE.md` symlink pointing to it: `ln -s AGENTS.md CLAUDE.md`
3. Do not duplicate root-level guidelines — directory files extend, not replace, the root
4. Follow the same quality bar and entry format as this file

**When updating an existing directory-level `AGENTS.md`:**
- Apply the same standards as updating this root file — fix stale info, add new pitfalls, update patterns
- If you worked inside a package and learned something, check its `AGENTS.md` before you commit

**When to use a directory file vs. adding here:**
- Knowledge that only matters when working *inside* that package → directory-level `AGENTS.md`
- Knowledge that affects how other packages *interact with* that package → root `AGENTS.md`
- Architectural patterns that span multiple packages → root `AGENTS.md`

### Quality Bar

Every entry should pass this test: *"Would this save a future agent at least 5 minutes of confusion or debugging?"* If not, it's not worth adding.

---

## Go Best Practices

### Code Formatting

All Go code must be properly formatted before committing:

```bash
# Format all Go files
gofmt -w .

# Or use goimports to also organize imports
goimports -w .
```

- Run `gofmt -d .` to check for formatting issues without modifying files
- Imports should be organized in groups: standard library, external packages, internal packages

### Linting

Run static analysis before committing:

```bash
# Basic linting (always run)
go vet ./...

# If golangci-lint is available (recommended)
golangci-lint run
```

Address all linting warnings before committing code.

## Architecture & Design Principles

### Single Responsibility

Each package, type, and function should have one clear purpose:

- **Packages** - A package should represent a single concept (e.g., `config`, `worktree`, `tui`)
- **Types** - A struct should model one thing; avoid "god objects" that do everything
- **Functions** - A function should do one thing well; if it needs "and" in its description, consider splitting it

### Separation of Concerns

Keep different layers distinct:

- **Domain logic** should not depend on I/O or presentation
- **I/O operations** (file, network, process) should be isolated behind interfaces
- **TUI/CLI code** should be thin wrappers that delegate to business logic

### Modular Design

Prefer small, focused packages over large monolithic ones:

- Extract reusable logic into dedicated packages under `internal/`
- Use interfaces to define boundaries between packages
- Avoid circular dependencies—if package A imports B, B should not import A

### Dependency Injection

Design for testability by accepting dependencies rather than creating them:

```go
// Prefer: accepts dependencies
func NewManager(logger Logger, store Store) *Manager

// Avoid: creates its own dependencies
func NewManager() *Manager {
    logger := log.New(...)
    store := NewFileStore(...)
}
```

This makes code easier to test with mocks and more flexible to configure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iron-Ham/claudio](https://github.com/Iron-Ham/claudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
