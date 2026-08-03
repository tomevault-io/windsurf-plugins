---
trigger: always_on
description: This is the single source of truth for both human contributors and coding agents
---

# AGENTS.md

This is the single source of truth for both human contributors and coding agents
(Claude Code and others) working in this repository: the architectural overview,
development workflows, and coding conventions.

## What this is

`structalign` is a single-binary Go CLI that shows how a struct's fields could be
reordered to use less memory, as **human-readable** output: it prints the
reordered struct plus a diff for review, rather than editing files or emitting a
machine-applicable patch the way `fieldalignment -fix` / `-fix -diff` do. It also
has an `-inspect` mode that prints a struct's memory layout (offset/size/align/
padding per field).

The program is split into small, decoupled packages:

- `main.go` (module root) — a thin entrypoint: `os.Exit(app.New(os.Stdout, os.Stderr).Run(os.Args[1:]))`.
- `pkg/common` — the public **contracts**: data types (`Target`, `Finding`,
  `Layout`, `LayoutField`, `DiffStyle`, `Colorize`) and interfaces (`Loader`, `Aligner`,
  `Inspector`, `Sizes`). Kept out of `internal/` so mockery can generate mocks
  from a non-internal source.
- `internal/` — the implementations: `loader` (go/packages adapter), `align`
  (runs the analyzer → findings), `layout` (computes struct layouts), `sizes`
  (`go/types` sizing adapter), `textdiff` (go-udiff line diff), `match` (glob
  filtering), `structfilter` (generated-file and `cpu.CacheLinePad` predicates),
  `config` (.structalignrc and env var mapping),
  `ui` (the `Printer` — all rendering + color/width helpers), `app` (flag parsing
  + wiring). Plus `testutil` (in-process `Target` builder for tests) and `mocks`
  (mockery-generated, test-only).

`_example/types.go` is sample input used for manual testing; the leading
underscore makes the Go tool skip the directory, so it never enters `./...`.
The module path is `github.com/peczenyj/structalign`, and `main` is at the module
root, so the install target is `github.com/peczenyj/structalign@latest` → binary
`structalign`.

## Commands

The repo uses [Task](https://taskfile.dev); the `Makefile` is a thin delegator
(`make X` runs `task X`). Run `task --list` to see everything.

```
task build                 # -> ./structalign
task lint                  # golangci-lint v2 (lint + formatters: gofumpt/goimports/gci)
task test                  # gotestsum over all packages
task test -- -update       # regenerate golden fixtures (internal/ui/testdata/*.golden)
task smoke                 # run both modes against ./_example
task generate              # regenerate code (go generate) + mocks (mockery); subtasks generate:code / generate:mocks
task ci                    # full pre-push gate: tidy:check, lint, go-consistent, build, test, smoke
go run . [flags] [packages]                   # packages: ./..., import paths, dirs, files
```

`enumer` and `mockery` are code generators; `DiffStyle` and `Colorize`
(`pkg/common`) are enumer-generated `uint8` enums (`go generate ./pkg/common`
after changing their constants), and mocks come from `mockery`. Generated files (`*_enumer.go`,
`internal/mocks/*`) are committed — regenerate, never hand-edit.

Exit code is meaningful: diff modes exit **1** when any reordering is found
(CI-friendly), **0** when none; `-inspect` always exits 0.

## Core architecture

The key design decision (see the README "How it works"): this tool **does not
reimplement** the field-alignment algorithm. `internal/align` runs the unmodified
upstream `fieldalignment.Analyzer` and intercepts the `SuggestedFix` it already
produces. The pipeline, orchestrated by `app.Run`:

1. **`loader.Load`** resolves CLI args via `golang.org/x/tools/go/packages`
   (`./...`, import paths, dirs, and — via `normalizeArgs` — single `.go` files)
   into `[]common.Target`. A `Target` is a loader-agnostic view of one typed
   package (syntax, types, type info, sizes) — it hides `go/packages.Package`.
2. **`align.Findings`** runs the analyzer over a `Target` (wiring an
   `analysis.Pass` and satisfying the `inspect` pass with `inspector.New`) and
   returns `[]common.Finding` — plain data (original + proposed struct text +
   message), not rendered output. **`layout.Layouts`** is the parallel inspect
   path: it reads `Sizes.Offsetsof`/`Sizeof`/`Alignof` to produce `[]common.Layout`.
3. **`app.Run` collects** all findings (or layouts) across the scanned targets
   into one slice, then post-processes that slice in `app`: **filter**
   (`-threshold`, by absolute bytes saved), **sort** (`-sort`, largest-first —
   findings by savings, layouts by `Layout.Total`), and hand the result to
   **`ui.Printer`**, which renders (unified / side-by-side / proposed-only diff
   via `textdiff`, or annotated layout) to an `io.Writer`. With `-summary` (diff
   only) it then prints a one-line `Summary: N structs affected, M bytes saved total`.
   The savings metric is the shared `app.savings(common.Finding) int64` helper
   (used by sort, threshold, and summary). Because the logic packages return data
   and `ui` consumes it, rendering is testable by injecting findings — no
   analyzer, no toolchain.

Two **injectable wrappers** are the crux of the decoupling and testability:

- **`common.Sizes`** abstracts `go/types` sizing. Its method set matches

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peczenyj/structalign](https://github.com/peczenyj/structalign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
