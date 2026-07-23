---
trigger: always_on
description: F# web framework proving that HATEOAS, statecharts, and semantic discovery compose into a pit of success for hypermedia APIs.
---

# Frank

F# web framework proving that HATEOAS, statecharts, and semantic discovery compose into a pit of success for hypermedia APIs.

## Commands

```bash
# Build (DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1 required on nix-darwin due to ICU mismatch)
DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1 dotnet build Frank.sln

# Test (excludes sample apps)
DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1 dotnet test Frank.sln --filter "FullyQualifiedName!~Sample"

# Frank.Tests is NOT in Frank.sln — test separately
DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1 dotnet test test/Frank.Tests/

# Format check (Fantomas 7.0.5)
dotnet fantomas --check src/
```

Always run `dotnet build Frank.sln` and `dotnet test` before claiming work is complete.
**Sample E2E tests:** Check `find sample/ -name "test-e2e.sh"` and run when working on issues that affect sample behavior.

## Project Structure

- `src/` — Library projects, multi-target net8.0/net9.0/net10.0
- `test/` — net10.0 only; `Frank.Tests` NOT in `Frank.sln`
- `sample/` — Sample apps (Frank, Falco/Giraffe/Oxpecker variants, OpenApi, Datastar)
- `.claude/worktrees/` — Git worktrees for feature branches (gitignored)
- `hooks/` — Git hooks (Fantomas pre-commit, Entrie CLI)

Five shipping packages: `Frank`, `Frank.Auth`, `Frank.OpenApi`, `Frank.Datastar`, `Frank.Analyzers`.
v7.3.2 in-scope packages (`Frank.Semantic`, `Frank.Validation`, `Frank.LinkedData`, `Frank.Provenance`, `Frank.Discovery`, `Frank.Cli`) don't exist yet — created fresh per spec.

## Constitution (non-negotiable)

1. **Resource-Oriented Design.** Resources are the primary abstraction. `resource` CE is the central API. Hypermedia over static specs.
2. **Idiomatic F#.** CEs for config, DUs for choices, Option over null, pipeline-friendly, declarative over imperative.
3. **Library, Not Framework.** No view engine, ORM, or auth system. Compose with ASP.NET Core.
4. **ASP.NET Core Native.** Expose `HttpContext` directly. Don't hide the platform.
5. **Performance Parity.** No runtime overhead vs raw ASP.NET Core routing. Avoid allocations in hot paths.
6. **Resource Disposal Discipline.** All `IDisposable` values MUST use `use` bindings.
7. **No Silent Exception Swallowing.** Middleware MUST log via `ILogger`. No bare `with _ ->`.
8. **No Duplicated Logic.** Same function in 2+ modules → extract before merge.

## Code Discipline

### Holzmann Rules (9–15)

AI-generated code routinely violates these. Push back when it does.

9. **Keep It Linear.** Max two nesting levels. Flatten with early returns, pipelines, or extracted functions.
10. **Bound Every Loop.** Every loop, retry, poll, recursion needs an explicit cap with defined cap-hit behavior.
11. **One Function, One Job.** Describable without "and." Hard limit: 60 lines.
12. **State Your Assumptions.** Preconditions in code (`invalidArg`/`invalidOp`/`assert`/`failwith`) — not comments.
13. **Narrow Your State.** No module-level `mutable`. Pass dependencies explicitly.
14. **Surface Your Side Effects.** I/O and mutations obvious at the call site. Separate pure from effectful.
15. **One Layer of Indirection.** Max one layer of dynamic dispatch. Linear composition over decoded elegance.

Run `/discipline` to grade changed code against these rules.

### Karpathy Guidelines

- **Think before coding.** State assumptions explicitly. Surface tradeoffs. If multiple interpretations exist, ask — don't pick silently.
- **Simplicity first.** Minimum code that solves the problem. No speculative features, abstractions, or error handling for impossible scenarios.
- **Surgical changes.** Touch only what the request requires. Don't improve adjacent code or formatting. Mention unrelated dead code — don't delete it.
- **Goal-driven execution.** Define verifiable success criteria before starting. Multi-step tasks: state the plan with per-step verification checks.

Run `/karpathy-guidelines` to review code against these before sending.

### False-green patterns (hunt these in adversarial review)

Green tests that don't prove the claim. Each is RED-test-able — the fix has an observable failing test the shortcut lacks. Caught repeatedly across the #333 capstone:

- **String-strip / post-hoc hack** — relativizing/fixing output via `.Replace` at one endpoint instead of at source. Fix at source; test every serving path.
- **Hollow decoration** — declaring a prefix/field without the substance: `href-vars: ""`, an `@context` prefix present but terms never compacted, a `type=` param with no target. Assert the substance (compacted term present AND full IRI absent), not mere presence.
- **Over-broad `catch`** — a guard wrapping more than the failing operation (413 catch around the whole pipeline, not just the body read) → unrelated errors mislabeled. Wrap only the throwing step; add a test that a downstream error is NOT mislabeled.
- **Sample-vocab baked into a shipping library** — the sample's terms hardcoded in `src/` (e.g. `ttt` in a Frank.* package). Thread the app's declared vocab through config; test a non-sample app gets none of the sample's terms.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frank-fs/frank](https://github.com/frank-fs/frank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
