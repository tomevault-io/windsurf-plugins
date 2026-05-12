---
trigger: always_on
description: - Personal terminal diff viewer.
---

# dunk agent notes

## purpose

- Personal terminal diff viewer.
- Hard-forked from `hunk` and slimmed down: no daemon, no MCP, no broker — agent integration runs through `.dunk/comments.json` on disk.
- Product target is "modern desktop diff tool in a terminal", not a pager-style TUI.

## major dependencies

- [Bun](https://bun.sh) runtime and package manager
- [OpenTUI](https://github.com/anomalyco/opentui) React terminal UI framework
- [Pierre](https://www.npmjs.com/package/@pierre/diffs) diff engine and terminal renderer

## core principles

- **Strict types, zero `any`** - The type system is your first line of defense. Never use `any`; use `unknown` when the type is genuinely not known.
- **Minimal and simple** - Prefer the simplest solution that works. Three similar lines are better than a premature abstraction.
- **Self-documenting** - Precise naming and strong types replace most comments. Code should read like prose.
- **Fix errors, don't suppress** - Fix linter and type errors at the root. Never `@ts-ignore`; use `@ts-expect-error` with explanation only when truly unavoidable.
- **Performance-conscious** - Memoize expensive computations, avoid unnecessary recomputation, virtualize long lists.

## what this means

- A new developer can understand any component by reading its types and props
- Type errors caught at compile time never reach users
- Refactoring is safe because the compiler catches breakage
- Code reviews focus on logic and architecture, not formatting or type correctness

## architecture

```text
CLI input
  -> parse runtime + config-backed view options
  -> normalize into one Changeset / DiffFile model
  -> App shell coordinates state, layout, and review navigation
  -> pane components render review UI
  -> Pierre-backed terminal renderer draws diff rows
```

- CLI entrypoints: `diff`, `show`, `stash show`, `patch`, `pager`, `difftool`.
- All input sources normalize into one internal changeset model.
- Pager mode has two paths: full diff UI for patch-like stdin, plain-text fallback for non-diff pager content.
- View defaults are layered through built-ins, user config, repo `.dunk/config.toml`, command sections, pager sections, and CLI flags.
- Review comments live in one committed `.dunk/comments.json` per repo. Watch mode picks up changes so a coding agent and a human reviewer can ping-pong on the same review.
- Prefer one source of truth for each user-visible behavior. When rendering, navigation, scrolling, or note placement share the same model, derive them from the same planning layer rather than maintaining parallel implementations.
- When UI behavior depends on derived structure or metrics, make that structure explicit in helper modules and reuse it across rendering and interaction code instead of re-deriving it ad hoc in multiple places.
- If a new implementation makes an older path obsolete, remove the dead path instead of keeping two overlapping systems around.

## architectural rules

- Keep the app review-first: the main pane is a single top-to-bottom stream of all visible file diffs.
- The sidebar is for navigation. Selecting a file jumps to that file in the main review stream; it should not collapse the main pane to one file.
- Keep Pierre as the diff engine and renderer foundation. Do not switch the main renderer back to OpenTUI's built-in `<diff>` widget.
- Keep split and stack views terminal-native and driven from the same normalized diff model.
- Preserve mouse + keyboard parity for primary actions.
- Keep the chrome restrained: minimal borders, no top menu bar, no redundant metadata headers.

## component guidance

- `App` should remain the orchestration shell for app state, navigation, layout mode, theme, filtering, and pane coordination.
- Pane rendering should live in dedicated components.
- New UI work should extend existing components or add new ones, not grow `App` back into a monolith.
- Shared formatting, ids, and small derivations belong in helper modules, not repeated inline.
- Prefer one implementation path per feature instead of separate "old" and "new" codepaths that duplicate behavior.
- When refactoring logic that spans helpers and UI components, add tests at the level where the user-visible behavior actually lives, not only at the lowest helper layer.

## testing

- Colocate unit tests with the code they cover (`src/core/foo.ts` + `src/core/foo.test.ts`, `src/ui/AppHost.*.test.tsx`, `src/ui/lib/*.test.ts`).
- Put shared unit-test helpers in `test/helpers/`.
- Name test helpers so they explicitly include `Test` and are clearly test-only (`createTestDiffFile`).
- Use repo-level `test/` directories by intent:
  - `test/cli/` for black-box CLI contract coverage.
  - `test/pty/` for PTY-backed live UI integration tests.
  - `test/smoke/` for opt-in terminal transcript smoke coverage.

## code comments

- Add short JSDoc-style comments to functions and helpers.
- Add inline comments for intent, invariants, or tricky behavior that would not be obvious to a fresh reader.
- Skip comments that only narrate what the code already says.

## naming

- Prefer names that match the role the code plays in the product and architecture.
- Use `layout` for structural placement or arrangement data.
- Use `geometry` for aggregate spatial data used by rendering, scrolling, or interaction.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amix/dunk](https://github.com/amix/dunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
