---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## MANDATORY: Documentation Updates Required for Every Change

**CRITICAL REQUIREMENT:** Every code change MUST include updates to the relevant documentation files. This is NOT optional. Before completing ANY task, you MUST review and update the following documentation:

### Required Documentation Files

| File | When to Update |
|------|----------------|
| `PRD.md` | ANY feature change, new functionality, UI changes, acceptance criteria updates, roadmap changes, architecture changes, data model changes |
| `PRIVACY.md` | ANY change affecting user data, permissions, third-party APIs, data storage, or data transmission |
| `CHROMEWEBSTORE.md` | ANY user-facing feature change, permission changes, version releases, description updates |
| `README.md` | ANY architecture changes, file structure changes, new dependencies, API changes |

### Documentation Update Checklist

Before marking ANY task as complete, you MUST verify:

1. **PRD.md**
   - [ ] Updated implementation status checkboxes
   - [ ] Updated feature descriptions if behavior changed
   - [ ] Added new features to the appropriate phase/section
   - [ ] Updated acceptance criteria
   - [ ] Updated roadmap status if applicable
   - [ ] Updated data models if types changed
   - [ ] Updated architecture diagrams if structure changed

2. **PRIVACY.md**
   - [ ] Updated "Last Updated" date if any privacy-related changes
   - [ ] Updated permissions list if manifest.json changed
   - [ ] Added new third-party services if integrated
   - [ ] Updated data collection/storage sections if applicable

3. **CHROMEWEBSTORE.md**
   - [ ] Updated "Last Updated" date if any user-facing changes
   - [ ] Updated feature list in descriptions
   - [ ] Updated permissions justification if manifest.json changed
   - [ ] Added to Version History for releases

4. **README.md**
   - [ ] Updated architecture sections if structure changed
   - [ ] Updated file structure if files added/removed/moved
   - [ ] Updated dependencies if package.json changed

### Enforcement

- **DO NOT** complete a task without reviewing these documentation files
- **DO NOT** assume documentation is already up to date
- **ALWAYS** read the relevant documentation files before making changes
- **ALWAYS** update documentation in the SAME commit as the code changes
- If you are unsure whether documentation needs updating, UPDATE IT

This requirement exists because documentation drift causes significant maintenance burden. Every change, no matter how small, must keep documentation in sync.

---

## MANDATORY: Git Commits After Major Changes

**CRITICAL REQUIREMENT:** You MUST create a git commit after every major change. Do not batch multiple unrelated changes into a single commit.

### When to Commit

Create a commit after completing any of the following:

- **Feature implementation**: After completing a new feature or significant functionality
- **Bug fix**: After fixing a bug (include the bug description in the commit message)
- **Refactoring**: After completing a refactoring effort
- **Documentation updates**: After updating documentation files
- **Configuration changes**: After modifying build config, manifest.json, or other config files
- **Test additions**: After adding or updating tests

### Commit Message Guidelines

Write clear, descriptive commit messages that explain **what** changed and **why**:

1. **Use conventional commit format**:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `refactor:` for code refactoring
   - `docs:` for documentation changes
   - `test:` for test additions/changes
   - `chore:` for maintenance tasks (deps, config, etc.)

2. **First line**: Short summary (50 chars or less), imperative mood
   - ✅ `feat: add notebook export functionality`
   - ❌ `added notebook export` or `adding export feature`

3. **Body** (when needed): Explain the "why" and any important details
   - What problem does this solve?
   - Why was this approach chosen?
   - Any side effects or areas affected?

### Example Commit Messages

```
feat: add source drag-and-drop reordering

Allow users to reorder sources within a notebook by dragging.
Uses HTML Drag and Drop API for modern browser support.
```

```
fix: prevent duplicate sources when adding from bookmarks

Sources were being duplicated when the same bookmark was added
multiple times. Now checks for existing URL before adding.
```

```
refactor: extract permission logic into shared utility

Moves permission checking code from sidepanel to src/lib/permissions.ts
for reuse across components.
```

### Enforcement

- **DO NOT** accumulate multiple unrelated changes before committing
- **DO NOT** use vague messages like "updates" or "fixes"
- **ALWAYS** commit working code (ensure build passes before committing)
- **ALWAYS** include related documentation updates in the same commit as code changes

---

## Project Overview

FolioLM (https://foliolm.com) is a browser extension that helps users collect sources from tabs, bookmarks, and history, then query and transform that content (e.g., create quizzes, podcasts, summaries).

## Browser Support


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulKinlan/NotebookLM-Chrome](https://github.com/PaulKinlan/NotebookLM-Chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
