---
trigger: always_on
description: > **Project:** VS Code WYSIWYG Markdown Editor (medium.com-style reading/writing experience)
---

# Instructions for AI Agents

> **Project:** VS Code WYSIWYG Markdown Editor (medium.com-style reading/writing experience)
>
> **Status:** MVP ~85% complete | **Core value:** Write markdown naturally—focus on content, not syntax
>
> **Source of Truth (order):** Code + tests → plan file (`roadmap/pipeline/*.md` or `roadmap/shipped/*.md`) → these instructions. Prefer source over docs when in doubt.

**Start here (quick boot-up):**
1) Open the current task file in `roadmap/pipeline/` and skim sections 2–4.  
2) Read `vibe-coding-rules/env-context.md` for architecture/perf budgets.  
3) Scan the code/tests referenced in the task with `rg` before writing anything.

---

## Critical Constraints

### 1. Reading Experience is PARAMOUNT
- Typography and readability > feature completeness
- Serif body text (prose, not code)
- Generous spacing (white space is a feature)
- **Test every change by reading a 3000+ word doc for 10+ minutes**

### 2. Test-Driven Development (MANDATORY)
**RED → GREEN → REFACTOR → VERIFY**

- Write failing tests BEFORE implementation
- A plan is only "done" when ALL tests pass (new + existing)
- Document tests in the plan file or test files
- See: [vibe-coding-rules/testing.md](vibe-coding-rules/testing.md)

### 3. Performance Budgets
- <500ms editor initialization
- <16ms typing latency (keystrokes)
- <50ms other interactions (cursor, formatting)
- <300ms menu/toolbar actions
- Handle 10,000+ line documents smoothly

### 4. VS Code Integration
- Deep Git integration (diffs, commits work correctly)
- Command palette commands (discoverable)
- Follow VS Code keyboard conventions
- Inherit theme colors (no hard-coded values)

### 5. Git & File Rules
- **Never commit or push** — User must review first
- **Use `git mv`** for renaming/moving tracked files (preserves history)
- **Pre-commit hook** — Automatically runs `npm run lint:fix` before each commit (see `.github/hooks/pre-commit`)

---

## Key Technical Decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Editor Provider | `CustomTextEditorProvider` | Text-based, VS Code handles save/undo, better Git |
| Editor Framework | TipTap (over raw ProseMirror) | Easier API, rich extensions, markdown built-in |
| Sync Debounce | 500ms | Balance responsiveness vs. performance |
| Document Sync | Full replacement | Simpler, VS Code handles internal diffing |
| Body Font | Serif (Charter/Georgia) | Prose, not code; matches premium editors |

---

## Detailed Guides (Load When Needed)

| Guide | When to Load | Load Proactively? |
|-------|--------------|-------------------|
| [common-pitfalls.md](vibe-coding-rules/common-pitfalls.md) | Known issues and solutions | Read before any task |
| [testing.md](vibe-coding-rules/testing.md) | Writing tests, TDD workflow, bug fixes | For all bug fixes |
| [vscode-integration.md](vibe-coding-rules/vscode-integration.md) | Commands, webview, extension patterns | For extension features |
| [ux-principles.md](vibe-coding-rules/ux-principles.md) | Response times, errors, accessibility | For UI changes |
| [styling.md](vibe-coding-rules/styling.md) | CSS, visual feedback, theme-aware styling, Apple-like polish | For CSS/styling changes |
| [coding-standards.md](vibe-coding-rules/coding-standards.md) | TypeScript style, code documentation, file organization | For all code changes |
| [image-and-dom-handling.md](vibe-coding-rules/image-and-dom-handling.md) | Images, Enter key, NodeViews, Markdown serialization | For image/DOM changes |
| [performance.md](vibe-coding-rules/performance.md) | Bundle size, startup, memory optimization | Reference if concerns arise |

**Usage:** Proactively load guides when starting relevant work. The guides contain detailed patterns and examples.

---

## Plan Workflow (TDD Enforced)

**For new features/plans:**

1. **Create Plan** → Use [`roadmap/task-plan-template.md`](roadmap/task-plan-template.md) as starting point (can use any AI coding tool or manually)
2. **Move to Pipeline** → Once plan is locked and ready: `git mv [source]/[name].plan.md roadmap/pipeline/[name].md`
3. **Write Tests First** → Create failing tests that define expected behavior
4. **Implement** → Write simplest clean solution to make tests pass
5. **Refactor** → Clean up while keeping tests green
6. **Verify** → `npm test` - ALL tests must pass
7. **Self-Review** → Audit your own changes (see checklist below)
8. **Ship** → `git mv roadmap/pipeline/[name].md roadmap/shipped/`

**Critical Rules:**
- Tests MUST be written BEFORE implementation (TDD)
- A plan is only `done` when ALL tests pass (not just new ones)
- **Self-review before shipping** - catch overlooked issues
- If issues found, debug/audit code - NO quick hacks or patches

---

## Feature Checklist

**Plan Management:**
- [ ] Plan file created and moved to `roadmap/pipeline/[name].md` when ready
- [ ] Update plan with implementation progress
- [ ] Document decisions and test approach in plan

**Testing (MANDATORY - TDD Required):**
- [ ] **Write failing tests FIRST** (`src/__tests__/`)
- [ ] Verify tests fail (confirms tests work)
- [ ] **Implement feature** to make tests pass
- [ ] **Run `npm test`** - ALL tests pass (new + existing)
- [ ] Cover positive, negative, edge cases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [concretios/markdown-for-humans](https://github.com/concretios/markdown-for-humans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
