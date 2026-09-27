---
trigger: always_on
description: **Scope**: A binding contract for both AI Agents and human maintainers working on the `zest-ssg/zest` repository (.NET 10+).
---

# Zest SSG Engineering Contract
**Version**: 3.1
**Scope**: A binding contract for both AI Agents and human maintainers working on the `zest-ssg/zest` repository (.NET 10+).
**Goal**: Preserve the C#/F# architecture boundary while producing code that is strictly conventional, grammatically precise, and genuinely readable.

---

## 0. Prime Directive: Readability Serves the Reader

Every rule below exists to reduce comprehension cost. When a rule fights readability, readability wins, and the deviation must be justified in a single comment.

- Consistency is a means, not an end.
- Never rename something that is already clear merely to satisfy a pattern.
- Test: can a new maintainer understand this unit within 30 seconds? If not, fix clarity before enforcing style.

---

## 1. Architecture Boundary (Non-Negotiable)

| Layer | Language | Responsibility |
| :--- | :--- | :--- |
| CLI, infrastructure, I/O, composition root | C# | User-facing entry points, filesystem, processes, configuration loading |
| Engine, DSL, domain logic, pure functions | F# | Template rendering, parsing, build pipeline, immutable data flow |

Rules:
- No cross-layer calls. F# must not reference C# CLI types, and C# must not reference F# engine internals.
- Cross-boundary data uses explicit DTOs. Never use anonymous types or `dynamic` across the boundary.
- Prefer LINQ in C#. Prefer `|>`, `List`, and `Seq` in F#. Avoid `for` and `while` unless performance evidence demands otherwise.

---

## 2. Naming Convention

### 2.1 Canonical Form: Two Words
- File, directory, type: `PascalCase` → `TemplateRenderer`, `PageBuilder`
- Method, variable: `camelCase` → `renderTemplate`, `cacheBuffer`

### 2.2 Semantic Structure
- First word: the domain object (`Template`, `Config`, `Build`, `Style`).
- Second word: the action or role (`Renderer`, `Parser`, `Compiler`, `Loader`).

### 2.3 Permitted Exceptions
Only **framework-mandated names** may violate the two-word rule. Every other exception is rejected.

Allowed:
- `Program.fs`, `Startup.cs`, `App.razor`, `Main`, `Index` — dictated by .NET, ASP.NET, or build tooling.
- Language-mandated constructs: `module`, `namespace` keywords aside, F# `Program` entry module.

Rejected:
- Community-convenient single words (`Router`, `Lexer`, `Parser`) are **not** exceptions. Rename them.
  - `Router` → `RequestRouter`
  - `Lexer` → `TokenScanner`
  - `Parser` → `TemplateParser`
- "It's already used elsewhere" is not an exception. Fix the usage.

Each permitted exception **must** carry a one-line justification:

```csharp
// Framework-mandated: ASP.NET Core requires the Startup class name.
public sealed class Startup { ... }
```

### 2.4 Forbidden Patterns
- Vague nouns: `Utils`, `Helper`, `Manager`, `Data`, `Common`, `Misc`, `Core`.
- Casual abbreviations: `Tmp`, `Cfg`, `Req`, `Mgr`, `Svc`, `Impl`.
- Vague verbs: `process`, `handle`, `get`, `do`, `run`. Use specific verbs: `parseContent`, `fetchMetadata`, `compileStyle`.
- Three-word-or-longer combinations: `TemplateRenderEngine` → `TemplateRenderer`.

### 2.5 Rename Discipline
- **New code**: fully compliant.
- **Touched files**: rename only when the current name is genuinely harmful. A clear legacy name beats an awkward new one.
- **Pure renames**: one dedicated commit. Never mixed with logic changes.
- **No rename without a reason.** If you cannot articulate the reason in one sentence, do not rename.

### 2.6 Reference Table

| Context | Forbidden | Required | Framework Exception |
| :--- | :--- | :--- | :--- |
| File | `Render.fs` | `TemplateRenderer.fs` | `Program.fs` |
| Type | `Builder.cs` | `PageBuilder.cs` | `Startup`, `Main` |
| Directory | `Zcss/` | `StyleCompiler/` | `Properties/` |
| Variable | `temp` | `cacheBuffer` | `i` (short loop index) |
| Module | `Utils` | `PathResolver` | — |

---

## 3. Documentation and Comments

### 3.1 File Header
Every non-trivial file must state its responsibility, dependencies, and any non-obvious invariant. Do not write a decorative one-liner.

```fsharp
// TemplateRenderer.fs
//
// Compiles Nunjucks templates and caches parsed results in memory.
// Caching prevents redundant disk reads on every page render.
//
// Invariant: cache keys are absolute, normalized paths.
// Callers must pass paths produced by PathResolver.
//
// Dependencies: Zest.Engine.Domain, System.IO
```

### 3.2 Public API: XML Documentation
Required for every public type and member in both C# and F#.

Cover:
- **Intent**: what the caller achieves, not how it is implemented.
- **Contract**: preconditions, postconditions, invariants.
- **Failure modes**: what happens on invalid input, and why that behavior was chosen.
- **Thread safety**: state it explicitly when relevant.

```fsharp
/// <summary>
/// Renders a template with the supplied context data.
/// Returns an empty string for invalid paths so a single
/// broken template cannot fail the whole build pipeline.
/// </summary>
/// <param name="templatePath">Absolute, normalized path to the .njk file.</param>
/// <param name="context">Data bag used for variable interpolation.</param>
/// <returns>The rendered output, or an empty string on path failure.</returns>
let renderTemplate templatePath context = ...
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zest-ssg/zest](https://github.com/zest-ssg/zest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
