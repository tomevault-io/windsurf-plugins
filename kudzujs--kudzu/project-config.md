---
trigger: always_on
description: - Kudzu's primary product goal is AI-assisted migration of ordinary React-shaped TSX to static deploy artifacts.
---

# Kudzu Project Rules

## Product Direction

- Kudzu's primary product goal is AI-assisted migration of ordinary React-shaped TSX to static deploy artifacts.
- Ordinary common React-shaped TSX should migrate with minimal source restructuring. Prefer compiler specialization over asking applications to replace declarative components, collections, hooks, or conditions with imperative DOM code; this direction is product-wide, not tied to Stay or any other application.
- Preserve familiar function components, props, children, JSX, hooks, and event-handler syntax where Kudzu can compile them safely.
- The output remains pre-rendered HTML, CSS, and only the route-specific ESM capabilities actually used. Static pages must ship no JavaScript.
- React, a VDOM, hydration, and a retained browser component tree remain forbidden.
- A small compiler-generated capability module is not a general client runtime. Route-specific effects and runtime path/query-parameter readers are allowed when the browser is the only place the value can exist.
- Native document navigation is the default. Do not add an SPA router unless a real migration fixture proves it necessary and the user explicitly approves it.
- Migration input may use a named or aliased `Link` import from `react-router-dom` with exactly one static root-relative `to` and native anchor props. The compiler prefixes `base`, lowers it to `<a href>`, and erases the package import. Dynamic or relative destinations, `NavLink`, router-only props, default/namespace imports, and non-JSX uses remain unsupported.
- A named or aliased `useParams` import from `react-router-dom` may be called directly without runtime arguments on a bracket page exporting `runtimeParams = true`. The compiler redirects it to Kudzu's existing pathname reader and erases the package import; build-known `getStaticPaths()` routes continue to use props.
- A named or aliased `useMatch` import from `react-router-dom` may directly initialize one top-level `const` from one exact static root-relative string pattern in route scope. Kudzu evaluates it case-insensitively from the build-known application route and erases the package import, adding no browser capability. Layout use, runtime-parameter pages, params, wildcards, query/hash patterns, trailing slashes, dynamic patterns, indirect calls, and pattern objects remain unsupported.
- A named or aliased `useSearchParams` import from `react-router-dom` may initialize one top-level `const [params]` or `const [params, setParams]` tuple. Direct top-level `const value = params.get("literal")` reads lower to nullable route signals initialized from `location.search`; the exact pagination form `const page = Number(params.get("literal")) || finiteNumber` and exact string fallback `const order = params.get("literal") || importedImmutableArray[staticIndex]` lower through the same signal and existing primitive binding evaluator. Direct setter calls from nested browser callbacks accept one synchronous inline updater over `URLSearchParams`; no options uses `history.pushState()`, while exactly `{ replace: true }` uses `history.replaceState()`. Dynamic names or indexes, direct-value setters, aliases, other methods on the outer params object, indirect reads, and broader wrappers remain unsupported.
- A named or aliased `useNavigate` import from `react-router-dom` may initialize one top-level `const navigate` binding. Direct calls from nested browser callbacks with one safe static root-relative destination lower to `location.assign()`; exactly `{ replace: true }` lowers to `location.replace()`. Dynamic or relative destinations, render-time calls, passed aliases, and other options remain unsupported.
- Before planning migration-related framework work, read `MIGRATION_ROADMAP.md`, `docs/next-architecture/0.9-semantic-compression.md`, `docs/next-architecture/0.9-implementation-plan.md`, `docs/next-architecture/0.9-baseline.md`, and `docs/next-architecture/0.9-benchmark-contracts.md`. Follow the first evidence-ready incomplete item and its session packet instead of inventing a new architecture.
- For every 0.9 semantic slice, begin with a real failing fixture, reduce it through native behavior, existing semantics, normalization, or an internal adapter in that order, and report semantic primitives, core passes/LOC, runtime concepts, browser bytes, fixtures, and benchmark deltas. A new semantic primitive requires architecture review after at least three unrelated real fixtures; fixture count does not itself authorize the primitive.

- New Kudzu source imports framework APIs from `@kudzujs/core`. Migration input may retain supported named hook imports from `react` plus default, namespace, or named `Fragment`; the compiler must erase those React module references and must never emit or execute React.
- Reduced Zustand migration syntax is contained by its compiler adapter and lowers to package-neutral `SharedStateIR` and `SharedActionIR` records before generic signal and handler consumers. Preserve existing Zustand source diagnostics and layout-lifetime behavior; do not expose a public store/adapter API or add a store runtime until an independent migration proves it necessary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kudzujs/kudzu](https://github.com/kudzujs/kudzu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
