---
trigger: always_on
description: doc.haus is an **open-source legal-agent platform built as a true fork of
---

# doc.haus

doc.haus is an **open-source legal-agent platform built as a true fork of
[OpenCode](https://github.com/anomalyco/opencode)**. It retargets OpenCode's agent
harness from code/git onto **legal documents** — the concepts map almost 1:1
(a *matter* is a project directory, a *document* is a file, a *conversation* is
a session). We fork rather than reimplement so we keep pulling upstream
innovation via `git merge upstream/dev`.

<principles>
- **Mergeability first.** Keep edits to upstream packages (`packages/core`,
  `server`, `llm`, `sdk`, `opencode`) at ~zero so upstream merges stay clean. A
  core edit is allowed only with a strong, written justification, isolated in one
  clearly-marked commit. Default to building elsewhere first.
- **Build additively.** All legal functionality lives outside upstream packages:
  - `dochaus/` — the legal config layer (provider/models in `opencode.json`,
    agents, the `search-document` tool, skills, commands). Loaded by pointing the
    server at it via `OPENCODE_CONFIG_DIR=<repo>/dochaus`, so the upstream
    `.opencode/` dev config is never touched.
  - `services/ingest/` — separate Bun+Hono service that creates matters and turns
    uploaded DOCX into embeddings (OpenCode has no upload endpoint and plugins
    cannot add HTTP routes, so ingestion must live out-of-band).
  - `apps/web/` — React+Vite frontend talking to the OpenCode SDK + ingest API.
- **Domain mapping is presentation-only.** Matter = project directory, Document =
  file. Relabel in the UI and our config; never rename core primitives.
- **No edge case handling, ever.**
</principles>

<runtime>
Three processes: (1) unmodified `opencode serve` (the engine), (2)
`services/ingest`, (3) `apps/web`. Provider is Google Vertex (Gemini) via ADC —
`gcloud auth application-default login` plus `GOOGLE_VERTEX_PROJECT` and
`GOOGLE_VERTEX_LOCATION`. Matters live under `WORKSPACE_ROOT`, independent of this
repo; the server scopes every session/tool to a matter via the
`x-opencode-directory` header.
</runtime>

---

The rest of this file is OpenCode's upstream contributor guide. It applies when
working inside upstream packages — follow it there to keep diffs mergeable.

---

- To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- The default branch in this repo is `dev`.
- Local `main` ref may not exist; use `dev` or `origin/dev` for diffs.

## Commits and PR Titles

Use conventional commit-style messages and PR titles: `type(scope): summary`.

Valid types are `feat`, `fix`, `docs`, `chore`, `refactor`, and `test`. Scopes are optional; use the affected package or area when helpful, e.g. `core`, `opencode`, `tui`, `app`, `desktop`, `sdk`, or `plugin`.

Examples: `fix(tui): simplify thinking toggle styling`, `docs: update contributing guide`, `chore(sdk): regenerate types`.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Do not extract single-use helpers preemptively. Inline the logic at the call site unless the helper is reused, hides a genuinely complex boundary, or has a clear independent name that improves the caller.
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Imports

- Never alias imports. Do not use `import { foo as bar } from "..."` or renamed imports like `resolve as pathResolve`.
- Never use star imports. Do not use `import * as Foo from "..."` or `import type * as Foo from "..."`.
- If a namespace-style value is needed, import the module's own exported namespace by name, for example `import { Project } from "@opencode-ai/core/project"`, then reference `Project.ID`.
- Prefer dynamic imports for heavy modules that are only needed in selected code paths, especially in startup-sensitive entrypoints. Destructure dynamic import bindings near the top of the narrowest scope that needs them so they read like normal imports. Avoid inline chains such as `await import("./module").then((mod) => mod.value())` or `(await import("./module")).value()`. Keep branch-specific imports inside the branch that needs them to preserve lazy loading.

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sure-scale/doc-haus](https://github.com/sure-scale/doc-haus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
