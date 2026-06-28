---
trigger: always_on
description: validates each manifest against the schema — so a passing `dotnet test` proves all templates compile
---

# CLAUDE.md

Guidance for Claude Code working in the **Koine** project. This file is authoritative for this
subdirectory; the workspace-level `C:\repo\POC\CLAUDE.md` applies on top of it.

## What this is

Koine is a **domain-specific language for Domain-Driven Design**. You write a bounded context's
ubiquitous language once in `.koi` files and the compiler emits idiomatic, self-contained C#
(value objects, entities, aggregates, invariants, commands, events, state machines, repositories,
the application/CQRS layer, context maps, etc.). C# is the primary, most complete target; TypeScript,
Python, and PHP emitters also ship, and the parser and semantic model are kept strictly target-agnostic
so a further emitter (e.g. Rust) is a new emitter, not a rewrite (`Emit/TypeScript/`, `Emit/Python/`, `Emit/Php/`).

Read `README.md` for the language overview and the full construct table, and `USER-STORIES.md` for the
roadmap (work is organized as releases **R1–R17**). The docs site source lives in `website/` (Astro
Starlight). Current version is **0.17.x** (set in `Directory.Build.props`).

## Commit identity (important)

Per the workspace rule, commit with the GitHub identity, not the work email:

```bash
git -c user.email=phmatray@gmail.com -c user.name="Philippe Matray" commit -m "..."
```

## Build, test, run

.NET 10. Solution is the modern `Koine.slnx`. From this directory:

> **Prerequisite:** `Koine.slnx` includes `src/Koine.Wasm` (browser-wasm RID), so a bare
> `dotnet build` / `dotnet test` over the solution needs the WebAssembly workloads. Install
> them once: `dotnet workload install wasm-tools wasm-experimental`. Without them the solution
> restore/build fails on the wasm project. (CI installs the same pair; see `.github/workflows/ci.yml`.)

```bash
./scripts/build/build.sh    # dotnet build && dotnet test (build.ps1 / build.cmd are equivalents)
dotnet build                # build only
dotnet test                 # run all tests (~1900)
dotnet test --filter "FullyQualifiedName~R9ValueObjectTests"   # a single test class

# Run the CLI
dotnet run --project src/Koine.Cli -- build templates/starters/billing/billing.koi --target csharp --out ./generated
dotnet run --project src/Koine.Cli -- build templates/starters/billing/billing.koi    # parse + validate only, no output
dotnet run --project src/Koine.Cli -- --version
```

`Directory.Build.props` sets `Nullable`, `ImplicitUsings`, `LangVersion latest`, `Deterministic`.
**`TreatWarningsAsErrors` is intentionally NOT set** — the generated demo code may emit warnings and
must still build, so don't add it.

### CLI commands (`src/Koine.Cli/Program.cs`)

`build` (compile/validate; `--target`, `--out`, `--glossary`, `--config`), `check` (model-versioning
compatibility against a `--baseline`), `fmt` (canonical formatter), `init` (scaffold), `watch`
(rebuild on change), `lsp` (language server over stdio), `mcp` (the MCP server — stdio by default, or
`--http [--port N] [--host H]` to serve it over HTTP for any client by URL; reuses `Koine.Mcp`'s
hosts). A path argument may be a single `.koi` file
**or a directory** — directory mode compiles every `.koi` under it as one model so cross-file
imports, context maps, and integration events resolve (R13/R14).

## Architecture (the pipeline is strictly layered — keep it that way)

```
.koi source
  → ANTLR lexer/parser   (src/Koine.Compiler/Grammar/KoineLexer.g4 + KoineParser.g4, visitor mode)
  → KoineModelBuilderVisitor (Parsing/) → semantic model (Ast/, NO C# concepts)
  → SemanticValidator (Semantics/) → diagnostics with line/column
  → IEmitter (Emit/CSharp/CSharpEmitter) → C# source files
```

The whole thing is orchestrated by `Services/KoineCompiler.cs`.

- **`Ast/`** is the target-agnostic semantic model (`SemanticModel`, `Nodes`, `Expressions`,
  `KoineType`, `ModelIndex`, `TypeResolver`). **No C#-specific concept belongs here** — that's the
  invariant that keeps multiple emitters possible.
- **`Semantics/`** holds the validators (`SemanticValidator` plus focused ones for CQRS, context
  maps, integration events, entity behaviors, expressions). Diagnostics carry source spans.
- **`Emit/CSharp/`** is the C# emitter, split by concern across partial classes
  (`CSharpEmitter.ValueObjects.cs`, `.Entities.cs`, `.Aggregates.cs`, `.Behaviors.cs`, `.Cqrs.cs`,
  `.Runtime.cs`). Supporting pieces: `CSharpTypeMapper`, `CSharpNaming`, `CSharpExpressionTranslator`,
  `UsingCollector`, `OperatorNeedsAnalyzer`, `CSharpEmitterOptions`. `Emit/Glossary/` emits the
  ubiquitous-language glossary; `Emit/TypeScript/`, `Emit/Python/`, and `Emit/Php/` are the additional language emitters.
- **`Services/`** also hosts the editor/tooling backend reused by `koine lsp`: `WorkspaceIndex`,
  `KoineLanguageService`, `SemanticTokenProvider`, `TokenLocator`, `RefactorService`,
  `CompatibilityChecker`.

The grammar is **split into a separate lexer grammar** so `matches /regex/` can use a lexer mode and
read a regex literal as one token without colliding with the `/` division operator. ANTLR sources are
generated at build time by `Antlr4BuildTasks` into `Grammar/gen/` — don't hand-edit generated parser
code; edit the `.g4` files.

## Testing stack & conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Atypical-Consulting/Koine](https://github.com/Atypical-Consulting/Koine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
