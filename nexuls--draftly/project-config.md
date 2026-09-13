---
trigger: always_on
description: Instruction set for AI agents working on **Draftly**.
---

# AGENTS.md

Instruction set for AI agents working on **Draftly**.
`CLAUDE.md` is a symlink to this file — edit this one.

---

## What Draftly is

A pluggable markdown editor and static previewer built on CodeMirror 6, published to npm
as `draftly`. The repo is a Turborepo monorepo managed with Bun.

Two ideas carry the whole design:

1. **The document is always plain markdown text.** There is no secondary document model.
   Richness is CodeMirror _decorations_ layered on top, which retract when the cursor
   enters a construct to reveal the raw syntax.
2. **A plugin owns a markdown feature end to end** — its parser extension, editor
   decorations, keymap, theme, _and_ its static HTML renderer. One class, one feature,
   both surfaces. This is what makes editor/preview parity structural rather than
   aspirational.

Read [`artifacts/architecture/overview.md`](artifacts/architecture/overview.md) before
your first substantive change.

---

## Start every session here

**Before touching any code, in this order:**

1. **[`artifacts/memory.md`](artifacts/memory.md)** — durable facts, traps that have cost
   time, the developer's preferences, and open questions. Non-negotiable; it exists
   precisely because the codebase's sharpest edges are invisible from a quick read.
2. **[`artifacts/tasks/index.md`](artifacts/tasks/index.md)** — what is in flight, what is
   blocked, what has shipped.
3. **[`artifacts/architecture/index.md`](artifacts/architecture/index.md)** — pick the
   documents relevant to your task from the "open it when" column. Do not read all of
   them by default.
4. **[`artifacts/repository-map.md`](artifacts/repository-map.md)** — if you need to
   locate something.

**Then load the relevant skill:**

| Working on                                                      | Read first                                                                                           |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Decorations, view plugins, facets, extensions, keymaps, widgets | `.agents/skills/codemirror/` — including `references/architecture.md` and `references/extensions.md` |
| `turbo.json`, workspaces, caching, CI, filtering                | `.agents/skills/turborepo/` — including the topic file under `references/`                           |

Both skills have `references/` subdirectories with far more depth than their `SKILL.md`.
Read the specific topic file, not just the summary.

**Non-negotiable, additionally:**

- Read `artifacts/architecture/plugin-table.md` **before any edit** to
  `plugins/table-plugin.ts`. Several of its mechanisms look like accidents and are not.

---

## Working rules

### 1. Ask when something is suspicious or conflicting

If the code contradicts an architecture document, a README claim, or a comment — **stop
and ask the developer.** Do not silently pick one and "fix" the other. The document may
be describing intent the code has drifted from, and that gap is information.

Six such conflicts are already logged in
[`artifacts/memory.md`](artifacts/memory.md#open-questions-for-the-developer). Add to that
table rather than resolving unilaterally.

### 2. Clean architecture, strictly

- **One feature, one plugin file.** Cross-plugin coupling is the thing this architecture
  exists to prevent.
- **`editor/` never imports `plugins/`.** Plugins are injected by the caller. Preserving
  this keeps the library tree-shakeable.
- **Keep pure logic pure.** Text utilities, parsing, and formatting must not acquire a
  CodeMirror dependency. This is why the table plugin's lowest layers are testable.
- **Layer boundaries are real.** Identify the layer you are editing and stay inside it.
- **~500 LOC is the plugin ceiling.** Split into a directory before exceeding it. Two
  files already exceed it and are tracked as debt, not precedent.

### 3. Modular by default

Prefer a new small module over growing an existing file. Prefer a pure function over a
method that reaches into state. Prefer injecting a dependency over importing a singleton.

### 4. JSDoc, strictly

Every exported symbol — class, function, type, interface, constant — carries JSDoc.
Follow the existing style in `editor/plugin.ts` and `editor/draftly.ts`.

```ts
/**
 * Build heading decorations by iterating the syntax tree.
 *
 * @param ctx - Decoration context with view and decoration array
 * @returns Nothing; decorations are pushed into `ctx.decorations`
 */
```

- Document **why**, not what the code already says.
- `@param` for every parameter, `@returns` where non-void, `@example` on public API.
- Non-obvious constants get a JSDoc line explaining the choice — especially
  `decorationPriority` values.
- Match the surrounding file's comment density. Do not add narration to code that reads
  clearly on its own.

### 5. Git history stays organised

**Commit only correlated edits. Never everything at once.**

- One logical change per commit. A refactor and a bug fix are two commits, even in the
  same file.
- Code and its artifact updates belong in the **same** commit — a doc that lands a commit
  later is a doc that will be forgotten.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexuls/draftly](https://github.com/nexuls/draftly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
