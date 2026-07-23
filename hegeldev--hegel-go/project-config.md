---
trigger: always_on
description: just build-libhegel     # Build libhegel.{so,dylib} in the sibling hegel-rust checkout
---

# Hegel for Go

## Build Commands

```bash
just build-libhegel     # Build libhegel.{so,dylib} in the sibling hegel-rust checkout
just test               # Run tests with coverage (fails if coverage < 100%)
just format             # Auto-format code
just lint               # Check formatting + linting
just docs               # Build API documentation
just check              # Run lint + check-docs + test (full CI check)
go test -run TestName ./...  # Run a single test
```

## What This Is

A Go implementation of the Hegel property-based testing library. Hegel is a
universal property-based testing protocol whose native engine ships as a Rust
shared library, `libhegel` (built from
[hegel-rust](https://github.com/hegeldev/hegel-rust)'s `hegel-c` crate).
hegel-go calls into libhegel directly via FFI using
[`github.com/ebitengine/purego`](https://github.com/ebitengine/purego).

## Architecture

The library is structured in layers, each building on the previous:

1. **FFI loader** (`internal/libhegel`) — purego-based dlopen of `libhegel.{so,dylib}`,
   one `*libhegel.Handle` struct with a `func`-typed field per C symbol, path
   resolution via the `HEGEL_LIBHEGEL_PATH` env var, otherwise a `go:embed`'d
   vendored binary (git-lfs) as the fallback.
2. **Test runner** (`runner.go`) — `testCase` implements
   `TestCase` by routing every operation (generate, span, target, collection,
   mark_complete) to one libhegel C call.
3. **Generators** (`generators.go`, `primitives.go`, `collections.go`,
   `combinators.go`, `composite.go`) — type-safe generator abstraction, span
   system, collection protocol. Generators take a `TestCase` and produce
   typed values; they never touch libhegel directly.

A conformance suite that validates generator output against the protocol is
on the roadmap — the previous Python-server pytest harness was removed in
the libhegel transition and needs to be replaced with one that doesn't
depend on a server emitting per-test-case status.

## Public API

The user-facing surface lives in `hegel.go` (canonical package doc). Entry points:

- `hegel.Test(t, fn, opts...)` — runs a property test against a `*testing.T`
- `hegel.Run(fn, opts...) error` — runs a property test outside `*testing.T`
  (conformance binaries, standalone tools); returns an error
- `hegel.MustRun(fn, opts...)` — like Run, but panics on failure
- `hegel.Draw(ht, gen)` — draws a value inside a test body
- `hegel.T` — passed to the [Test] body; methods include `Note`, `Fatal`, `Fatalf`
- Generators: `Integers`, `Floats`, `Text`, `Booleans`, `Lists`, `Maps`, ...
- Options: `WithTestCases(n)`, `WithDatabase(path)`, ...

## Testing Philosophy

- **100% per-file code coverage** is mandatory. `just check` fails if any
  file is uncovered. Use `// coverage-ignore` only for genuinely untestable
  code (e.g. platform-specific branches, fixed-shape CBOR encode that can't
  fail). The annotation count is ratcheted in
  `.github/coverage-ratchet.json` and cannot increase without justification.
- **Function-pointer stubs** are the primary way to cover libhegel error
  paths.
- **Use the real libhegel** for integration tests.

## Locating libhegel

The library is resolved in this order; first hit wins:

1. `$HEGEL_LIBHEGEL_PATH` if set (loaded directly; no embedded fallback if it
   fails to open)
2. the `go:embed`'d vendored binary for the host platform (see below),
   materialized to `~/.cache/hegel-go/libhegel/<version>/` and dlopen'd from
   there. Skipped when `$HEGEL_LIBHEGEL_PATH` is set or on a platform with no
   vendored artifact.

The library does **not** hunt for a sibling `hegel-rust` checkout; local
development against a fresh build goes through `HEGEL_LIBHEGEL_PATH` (see below).

`<ext>` is `so` on Linux, `dylib` on macOS, `dll` on Windows.

End users get a pre-compiled `libhegel` vendored inside the module: the
per-platform binaries live in `internal/libhegel/libs/` (stored via **git-lfs**)
and are `go:embed`'d by the per-platform `embed_*.go` files into
`embeddedLib`. Run `git lfs install` after cloning so the real bytes (not the
LFS pointer) are present at build time. The pinned version lives in
`internal/libhegel/version.go`; `just vendor-libhegel [version]` refreshes both
the binaries and that constant from a hegel-rust GitHub release.

For local development against an unreleased libhegel, point
`HEGEL_LIBHEGEL_PATH` at a local build. `just test` / `just check` do this for
you: they run `build-libhegel` (delegates to `cargo build --release -p
hegeltest-c` in the sibling `hegel-rust/` checkout) and export the resulting
`.so` path, so a fresh local build wins over the vendored copy. Pass `just test
vendored` to skip the local build and exercise the embedded binary instead.

## Tooling Choices

- **Go version**: the oldest version supported by go.dev (1.N-1); CI tests 1.N and 1.N-1
- **FFI**: `github.com/ebitengine/purego` — runtime dlopen, no cgo
- **Test framework**: `testing` (Go stdlib) — run via `go test -race -coverprofile=coverage.out -covermode=atomic ./...`
- **Linter**: `go vet` (stdlib) + `staticcheck` v0.7.0 (2026.1) — run via `just lint`
- **Formatter**: `gofmt` (bundled with Go) — check with `gofmt -l .`, apply with `gofmt -w .`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hegeldev/hegel-go](https://github.com/hegeldev/hegel-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
