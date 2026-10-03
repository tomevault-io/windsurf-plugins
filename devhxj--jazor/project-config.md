---
trigger: always_on
description: `Jazor.slnx` is the entry point for the .NET solution. The transformation branch has one active Razor-to-Vue direction:
---

# Repository Guidelines

## Project Structure & Module Organization

`Jazor.slnx` is the entry point for the .NET solution. The transformation branch has one active Razor-to-Vue direction:

| Line | Mode | Key Projects | Description |
|------|------|-------------|-------------|
| **Razor-to-Vue transformation** | Active | `Jazor.RazorVue`, `Jazor.Analyzer`, `Jazor.Compiler`, `Jazor.Emit` | Official Razor SG generated C# -> Roslyn `IOperation` -> Vue render-function `.mjs` |

Shared infrastructure used by the active line:

| Project | Role |
|---------|------|
| `src/Jazor.Compiler` | Core C#-to-JS compiler (`IOperation` -> Acornima ESTree), targets `netstandard2.0` |
| `src/Jazor.CLR` | CLR runtime shims with `[WhiteList]` declarations and JavaScript implementations |
| `src/Jazor.CLR/doc/` | Module docs co-located with CLR runtime source |
| `src/Jazor.Analyzer` | Static analyzer enforcing whitelist usage at compile time |
| `src/Jazor.Compiler.Generator` | Source generator scanning `[WhiteList]` attributes |
| `src/Jazor.CLR.Generator` | Type mapping and binding code generator |
| `src/Jazor.Emit` | Emit pipeline, bundle materialization, and SourceMap output |
| `src/Jazor.Common` | Shared contracts, naming, and symbol formatting utilities |
| `src/ECMAScript` | Core ECMAScript AST implementation |
| `src/ECMAScript.Contract` | ECMAScript contract definitions and attributes |
| `src/ECMAScript.WebIDL.Generator` | WebIDL spec-to-C# binding generator |
| `src/Jazor` | NuGet package bundling runtime, analyzer, generators, emit, and MSBuild integration |

ECMAScript ecosystem layer:

| Project | Role |
|---------|------|
| `src/ECMAScript.Vue` | Vue 3 core type bindings |
| `src/ECMAScript.VueContract` | Vue component contracts, descriptors, and slot metadata attributes |
| `src/ECMAScript.VueRoute` | Vue Router type bindings |
| `src/ECMAScript.Vuetify` | Vuetify component wrappers (props, events, slots, value types) |
| `src/ECMAScript.Pinia` | Pinia state management bindings |
| `src/ECMAScript.Style` | Strongly typed, deterministic CSS-in-JS authoring and runtime module |
| `src/ECMAScript.Lucide` | Supported tree-shakeable Lucide icon bindings |
| `src/ECMAScript.DateFns`, `VueUse`, `FloatingUi`, `VeeValidate`, `VueI18n`, `VueQuery`, `VueDraggable`, `FilePond`, `WangEditor`, `Monaco` | Ecosystem bindings published from preview.5; Support status still requires browser smoke and real RazorVue consumer evidence |

ASP.NET Core integration layer:

| Project | Role |
|---------|------|
| `src/Jazor.AspNetCore` | ASP.NET Core runtime integration |
| `src/Jazor.AspNetCore.Dev` | Development-time integration (HMR, DevServer bridging) |

Test projects live under `src/Jazor.CompilerTest`, `src/Jazor.CLR.Test`, `src/Jazor.RazorVue.Sg.Test`, `src/Jazor.EmitTest`, `src/ECMAScript.Style.Test`, `src/ECMAScript.WebIDL.GeneratorTest`, `src/ECMAScript.VueRoute.Test`, `src/ECMAScript.Pinia.Test`, and `src/ECMAScript.Pinia.Testing.Test`. Auxiliary tooling outside the main solution includes `samples/Wiki` and other projects under `samples/`.

Documentation is organized under `docs/` in five categories:
- `docs/01-overview/` — product scope, reading map, and system overview
- `docs/02-architecture/` — current architecture, module ownership, and stable boundaries
- `docs/03-guides/` — installation, configuration, authoring, development, and testing
- `docs/04-roadmap/` — current direction, status, and reproducible quality gates
- `docs/05-history/` — concise background for retired routes and major evolution

Documentation interpretation rule:
- Historical exploration, old audits, and fixed test snapshots belong only to `docs/05-history/evolution.md`; Git history retains the detailed record.
- For current compiler semantics, lowering direction, and support boundaries, prefer `src/Jazor.Compiler/ImplementationPrinciples.md`, `docs/02-architecture/compiler.md`, `docs/04-roadmap/current-status.md`, and the current `src/Jazor.Compiler/README.md` / `src/Jazor.CompilerTest/README.md`.

RazorVue artifact and lowering boundary rule:
- The production input is official Razor SG generated C#, and the output contract is a Vue render-function `.mjs` artifact. Razor DR/IR, generated SFC output, and Jolt protocols are not fallback paths.
- Razor/C# compiler already validates Razor-side unknown parameters, required parameters, and parameter type mismatches. RazorVue lowering should directly translate official SG generated C# and must not duplicate those checks.
- Do not introduce intermediate wrapper-JS marker protocols for RazorVue slot/template transport when the same behavior can be expressed as the final Vue render-function shape directly.
- When RazorVue lowering needs CLR-aware type mapping, import collection, symbol binding, reference stability, or other compiler-owned semantics, it must flow through `Jazor.Compiler` / `SemanticWalker` translation hooks rather than bypassing them with hand-assembled Acornima AST or ad hoc JavaScript string stitching.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devhxj/Jazor](https://github.com/devhxj/Jazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
