---
trigger: always_on
description: All documentation, comments, variable names, function names, and user-facing strings **must use British English spelling** (`optimise`, `behaviour`, `colour`, `initialise`, `serialise`, `catalogue`). Exceptions: protocol identifiers (LSP `initialize`), CSS/HTML properties (`color`), Go stdlib names.
---

# CLAUDE.md

## Language Convention

All documentation, comments, variable names, function names, and user-facing strings **must use British English spelling** (`optimise`, `behaviour`, `colour`, `initialise`, `serialise`, `catalogue`). Exceptions: protocol identifiers (LSP `initialize`), CSS/HTML properties (`color`), Go stdlib names.

## Core Principles (Non-Negotiable)

1. **If it compiles, it works.** The 2026-04-15 adversarial audit
   documented 23 counterexamples across P0 (soundness floor), P1
   (security), P2 (soundness cleanup), and P3 (tooling). All 23 are
   remediated with regression tests (see `docs/AUDIT_REMEDIATION.md`
   — completion marker landed 2026-04-16). The principle now holds
   for every path exercised by `cabal test`, `scripts/example-sweep.sh`,
   and `runtime-go/rt/*_test.go`. Residual P4 items (fully-typed
   codegen to eliminate `any` in emitted Go, and Sky-test harness
   port) remain as future-work tracked in `docs/PRODUCTION_READINESS.md`.
   Defence in depth (panic recovery + `Err` return at Task boundaries)
   remains the reliability floor under the v1.0 milestone.
2. **Dev experience is top priority.** Clear errors, predictable behaviour, no user-written FFI.
3. **Root-cause fixes only.** Fix at the correct abstraction layer. **Never suppress type errors or warnings.**
4. **Production-grade architecture.** Must scale to large Go packages (Stripe SDK). Must remain maintainable.

## Non-Regression Rules

These constraints are enforced by `sky verify`, `test/Sky/ErrorUnificationSpec.hs`, and the audit-remediation specs under `test/Sky/**`. Violating them breaks the repo:

- **No `Result String a`** in any public surface. Use `Result Error a`.
- **No `Task String a`** in any public surface. Use `Task Error a`.
- **No `Std.IoError`** — the pre-v1 error ADT was deleted.
- **No `RemoteData`** — the pre-v1 async-state type was deleted.
- **No runtime panic from well-typed Sky code.** Every known panic class has a regression test in `runtime-go/rt/*_test.go` or `test/Sky/**Spec.hs`.
- **No silent numeric coercion.** `AsInt` / `AsBool` / `AsFloat` returning zero on type mismatch is a violation of P0-2 (see `docs/AUDIT_REMEDIATION.md`). New code uses the fallible `AsIntChecked` variants; lenient display-only helpers carry the suffix `OrZero` so the intent is visible at every call site.
- **No raw `.(T)` assertions on generated any-typed thunks.** Typed codegen must route through a runtime `Coerce` helper (see P0-3). Grep gate: `sky-out/main.go` files contain no `any(body).(T)` patterns outside the `Coerce*` helpers.
- **Record field enumeration sorts by `_fieldIndex`.** Any `Map.toList fields` in codegen that feeds field order (struct decl, auto-ctor, destructure) sorts by declaration index before emission. Violating this swaps auto-ctor parameters (P0-4).
- **Secrets are typed.** `Auth.signToken` / `Auth.verifyToken` take `String`, not `any`. `fmt.Sprintf("%v", secret)` is forbidden — explicit `.(string)` assertion on a typed boundary, with minimum-length validation (P1-4).
- **`sky check` is a superset of `sky build`.** `sky check` runs the Go emitter and invokes `go build` on the output. If `sky build` would fail, `sky check` fails (P0-1). Regression test: for every fixture in `test-files/`, both commands agree on accept/reject.

## Testing Rules

- **Every new language feature or runtime helper needs a test.** Cabal specs for compile-time behaviour; `runtime-go/rt/*_test.go` for runtime helpers; `tests/**/*Test.sky` for stdlib semantics.
- **Every bug becomes a regression test** *before* landing the fix. The failing test is the discovery artefact; without it, the class comes back. Audit items specifically require the test to fail against HEAD~1 and pass at HEAD.
- **`sky test <file>` is the user-facing runner.** See `sky-stdlib/Sky/Test.sky` for the API.
- **Runtime verification on every push.** `sky verify` builds and runs each example, catching panics and HTTP failures that `--build-only` misses.

## Tooling Rules

- **CLI commands must be correct end-to-end.** `sky build` / `sky run` / `sky check` / `sky fmt` / `sky test` all auto-regen missing FFI bindings and propagate exit codes.
- **LSP capabilities must match `docs/tooling/lsp.md`.** If you add a capability, document it. If a feature is incomplete, narrow the claim — don't lie in docs.
- **Formatter must be idempotent.** Two passes produce byte-identical output. Fixtures in `test/Sky/Format/FormatSpec.hs` guard this.

## Effect Boundary: Task

ALL effectful operations flow through `Task`:
- **Pure** (`String.length`, `List.map`) — no wrapping
- **Fallible** (`String.toInt`, `Dict.get`) — `Result` or `Maybe`
- **Effectful** (`File.readFile`, `Http.get`, `println`) — `Task Error a`
- **Entry** (`main`) — may return `Task`; runtime auto-executes

FFI boundary mapping: Go `(T, error)` → `Result Error T` | Go `error` → `Result Error ()` | panics → `Err` | nil → `Maybe`/`Result`

## Environment Variable Precedence


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anzellai/sky](https://github.com/anzellai/sky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
