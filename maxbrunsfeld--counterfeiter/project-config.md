---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`counterfeiter` is a CLI (module `github.com/maxbrunsfeld/counterfeiter/v6`) that generates Go test doubles ("fakes") for interfaces, function types, and whole packages. It is typically invoked via `//go:generate` directives. Requires Go modules; CI runs on `stable` and `oldstable` Go on Linux and Windows.

## Commands

Full CI pipeline (vet → regenerate fakes → verify clean git tree → tests):

```shell
./scripts/ci.sh          # Linux/macOS
.\scripts\ci.ps1         # Windows
```

Individual steps:

```shell
go vet ./...
go generate ./...                  # regenerate all fakes under fixtures/ (directives use `go run`, so no install needed)
./scripts/checkclean.sh            # fail if regenerated fakes differ from committed ones
./scripts/cleanfakes.sh            # delete every */*fakes/fake*.go (then `go generate ./...` to rebuild)
go test -race . ./fixtures/...   # packages that exercise fakes or the generator concurrently
go test ./arguments/ ./command/ ./generator/ ./integration/
```

Run a single package's tests or a single spec. Tests use `sclevine/spec` + `gomega`; spec names are nested, so match with a regex on the top-level test function and the spec path:

```shell
go test ./generator/ -run TestGenerator
go test ./integration/ -run 'TestIntegration/round_trip_as_module/working_with_a_module'
go test ./arguments/ -run TestParsingArguments -v
go test ./command/ -run TestRunner
go test -run TestFakes .                           # generated_fakes_test.go at repo root
go test -race -run TestConcurrency .              # concurrency_test.go at repo root; only meaningful with -race
go test -bench . -benchmem .                       # benchmark_test.go at repo root
```

Debug env vars: `COUNTERFEITER_DEBUG=1` enables log output; `COUNTERFEITER_DISABLECACHE=1` bypasses the package-load cache; `COUNTERFEITER_PROFILE=1` writes `counterfeiter.profile`; `COUNTERFEITER_NO_GENERATE_WARNING=1` silences the "use -generate" warning. In tests, `log.SetOutput(io.Discard)` is set in the top-level test functions — comment it out to see generator logs.

## Architecture

Pipeline for one run (`main.go` is intentionally thin and should stay that way):

1. **`command.Detect`** (`command/runner.go`) turns the process into a list of `Invocation`s. In normal mode that is the single CLI invocation (it reads `GOFILE`/`GOLINE` from `go generate`). In `-generate` mode it scans every `.go` file in the cwd package for lines starting with `//counterfeiter:generate ` and builds one invocation per line. This is why `-generate` is much faster than many `//go:generate` lines: one process, one package load.
2. **`arguments.New`** (`arguments/parser.go`) parses each invocation's flags and positional args into `ParsedArguments`: source package dir, package path, interface name, fake name (`Fake` + exported interface name), output path (default `<pkgdir>/<pkg>fakes/fake_<snake_case>.go`), destination package name, and modes (`-p` package mode, `-` print to stdout, `-q`, `-header`). A `-header` on the top-level `-generate` line is inherited by directives that lack one (handled in `main.go`).
3. **`generator.NewFake`** (`generator/fake.go`) loads packages with `golang.org/x/tools/go/packages` (`loader.go`), finds the target `types.TypeName` (`findPackage`), and populates the `Fake` struct: `Methods` (from `interface_loader.go` / `package_loader.go`) or a single `Function` (`function_loader.go`), `Params`/`Returns` (`param.go`, `return.go`), and `Imports`.
4. **`Fake.Generate`** executes one of three `text/template`s — `interface_template.go`, `function_template.go`, `package_template.go` — then runs `goimports` (`imports.Process`) on the output. `main.go` runs `go/format` again and writes the file.

Key supporting pieces:

- **`generator.Imports`** (`import.go`) dedupes imports by package path and guarantees unique aliases (appends `a`, `b`, … on collision). `addImportsFor` in `loader.go` walks `types.Type` recursively to collect every package a fake needs; add a case there when a new `types.Type` kind shows up (it logs `!!! WARNING: Missing case`).
- **Generics**: `findPackage` / `getGenericTypeData` (`loader.go`) extract type params/constraints into `GenericTypeParameters*` strings used by the template. The compile-time assertion for a generic fake is emitted inside a blank generic func (`func _[T C]() { var _ pkg.I[T] = new(FakeI[T]) }`) so any constraint kind works. A target that is itself a constraint interface (unions or `~T`) is rejected up front because it cannot be implemented.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxbrunsfeld/counterfeiter](https://github.com/maxbrunsfeld/counterfeiter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
