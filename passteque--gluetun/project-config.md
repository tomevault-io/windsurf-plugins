---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS

Guidance for coding agents working in this repository.

## Scope and priorities

- Keep changes minimal and targeted. Feel free to do light refactors that are relevant to the modifications.
- Breaking changes:
  - Do not introduce breaking usage behavior (cli flags, environment variables, etc.) unless explicitly agreed.
  - Do not introduce breaking changes for the Go API in the `pkg/` directory.
  - If a compatibility break seems beneficial, stop and ask for confirmation before implementing it.
- Update or add tests when behavior changes.

## Go coding conventions

### General guidelines

- Use explicit, descriptive variable names by default.
  - Notable bad examples: `req`, `resp`, `cfg`, `v`
  - Allowed short-name exceptions:
    - indexes such as `i`, `j`
    - `ctx` for `context.Context`
    - `t` for `*testing.T` and `b` for `*testing.B`
    - `ctrl` for `*gomock.Controller`
    - `err` for `error`, `errs` for `[]error`
    - `wg` for `*sync.WaitGroup`
- Avoid using global variables except for:
  - exported sentinel errors that are used outside the package boundaries
  - regular expressions defined with `regexp.MustCompile`
  - variables set by the build pipeline, such as `Version` and `BuildDate`
- Constants
  - Prefer defining them inline in a function if it's only used in that function, rather than at the package level.
  - Each one should be defined right above where it's used, instead of having multiple defined at the same place in a `const ()` block
  - If one is only used in a single production code function, define it right above it so it's more local for readability.
  - Do not define constants when constants exist in other packages, for example `http.StatusBadRequest` or `log.LevelDebug`.
- Structs
  - Prefer defining them inline in a function if it's only used in that function, rather than at the package level.
- Do not use the short if form, prefer the longer one
- Follow modern Go, according to the Go version defined in go.mod. Prefer modern constructs when equivalent:
  - Example: use `for i := range 5` rather than `for i := 0; i < 5; i++`.
  - Example: use `new("string")` rather than helper wrappers such as `stringPtr("string")`.
  - Example: no need to pin variables in for loops when using them in goroutines or subtests.
- Use `New(...) *Item` constructor per package. Each package should ideally only have one constructor, although this is not a strict rule. The constructor should return a pointer to the struct, and not an interface.
- Always prefer using context-aware functions, for example:
  - `exec.CommandContext` rather than `exec.Command`
  - `http.NewRequestWithContext` rather than `http.NewRequest`
- Never export a symbol unless absolutely necessary.
- Always use the most restrictive builtin types. For example prefer `uint` over `int` if it's only zero or positive. Prefer `uint16` is the max value is 65535.
- Prefer using builtin types whenever possible AND do not define single field structs unless necessary
- Prefer splitting a code line only when it triggers the `lll` linter, do not split a command or arguments list for each element
- Use `netip` types instead of `net` types whenever possible
- Use constants instead of variables whenever possible, especially function-local inline constants.
- Prefer using pure functions over methods when possible. Especially if the method does not need any fields from the receiving struct, it should be a pure function.
- Do not use `time.Sleep`, prefer using a `time.Timer` with a `select` statement also listening on a context cancelation
- `panic`:
  - should only be used when a programming error is encountered and you should NOT return errors for programming errors (such as passing nil objects)
  - Its counterpart `recover` should not really be used, except for testing a panic in test code (or use `assert.PanicsWithValue`).

### Directory structure and file naming

- Executable main packages with a single `main()` function must be in the `cmd` directory.
  Prefer having top level logic and have a longer `main()` function rather than having an `internal/app` package.
- Code lives by default in subpackages within the `internal` directory
- Code needing to be imported by external Go modules must be in subpackages within the `pkg` directory
- Example code especially using the `pkg` directory must be in `main` packages within the `examples` directory, each with a single `main.go` function.
- If AND only if the repository is a Go library and not a Go application, you may have Go files at the root of the project to simplify import paths. Most of the code should still be in subpackages in the `internal` directory.
- Interfaces should be defined in `interfaces.go` files for each package. If there are unexported interfaces which need to be mocked, which is rare, they should be defined in `interfaces_local.go` files.
- Mock files are
  - `mocks_generate_test.go` which only contains `//go:generate` directives for generating mocks, and no actual code
  - `mocks_test.go` which contains the generated mocks from exported interfaces and no other code, and is ignored in coverage reports
  - `mocks_local_test.go` (rare) which contains the generated mocks from unexported interfaces and no other code, and is ignored in coverage reports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [passteque/gluetun](https://github.com/passteque/gluetun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
