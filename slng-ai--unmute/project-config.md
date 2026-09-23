---
trigger: always_on
description: Go CLI that compiles a declarative voice-agent spec into orchestrator-native artifacts. `docs/ARCHITECTURE.md` explains the design and points at the load-bearing code. Go structs and `internal/target` own machine behavior; `docs-site/` is the public user guide. Local feature work lives in ignored `specs/<nnn>-<slug>/` directories.
---

# Unmute CLI

Go CLI that compiles a declarative voice-agent spec into orchestrator-native artifacts. `docs/ARCHITECTURE.md` explains the design and points at the load-bearing code. Go structs and `internal/target` own machine behavior; `docs-site/` is the public user guide. Local feature work lives in ignored `specs/<nnn>-<slug>/` directories.

## The one rule
Unmute is written in Go, so you maintain **Go code**, but you also write some Python code snippets and examples, in Python. Checked-in Python has to pass `ruff check .` (CI enforces it). Run `ty check` too when you have the provider SDKs installed, otherwise it only reports imports it cannot resolve.

## Tooling
- Go 1.26 (pin in `go.mod`, and keep it on a Go line that still gets security patches, which is the newest two); `CGO_ENABLED=0` static binary; version stamped at link time, never hardcoded.
- Direct deps — `cobra`, `goccy/go-yaml` (gives line/col on parse errors), `google/jsonschema-go` (**v0.x — pin the exact version, bump deliberately**), and the Charm TUI stack: `charmbracelet/bubbletea` + `bubbles` + `lipgloss` power the interactive console (custom MVU styled with Lip Gloss), while `charmbracelet/huh` v1.0.0 is scoped to the accessible/headless renderer only. **The interactive path imports no `huh`; Lip Gloss is expected there. All color lives in `internal/style` — no color literal anywhere else** (guarded by `internal/style/literal_test.go`, which walks every Go string literal in the tree). Everything else is stdlib. **No new dep for what a few lines of stdlib do — justify any addition in the PR.** No `viper` until a real global config file exists.
- `golangci-lint` from day one (`.golangci.yml`).
- Make targets: `build test smoke contracts lint fmt install`. `contracts` re-fetches the published SLNG conformance fixtures and fails on a digest mismatch: network, no accounts.

## Command rules (cobra) — these are what make commands testable, not suggestions
1. Build the tree with a `newRootCmd()` constructor; **no package-level `var rootCmd`** (fresh tree per call = flag isolation between tests).
2. Write to `cmd.OutOrStdout()` / `cmd.ErrOrStderr()`, **never `fmt.Println`** (a stray Println is invisible to tests — it's a bug).
3. `RunE`, never `Run`. **No `os.Exit` / `log.Fatal` inside a command** — return errors wrapped with `%w`.
4. `os.Exit` lives only in `main.go`. `Execute()` builds the root, prints any error once to stderr, returns the exit code.
- `SilenceUsage` + `SilenceErrors` on the root. Exit codes: `0` ok, `1` error — add more only when a consumer actually reads them. Warnings → stderr + exit 0 (never a silent downgrade).

### A command says what it did, and what somebody has to fix
Nothing else. `unmute --version` is one line, `validate` is its result rows,
`compile` is the list of files it wrote, and `init` is the list of files it
created. A warning, a prerequisite or an error is added to that; a description
of the output is not, because the output is on disk and
`build/<target>/compile-report.json` is next to it.

This is not taste, it is what makes the warnings readable. Every forwarded
binding, derived worker count and resolved route used to print on every run
alongside eight advisory notes about how a framework works, so twenty-odd lines
of nothing-to-do buried the two lines that said a secret was undeclared and a
tool call would 400. **A new line on stdout has to name something the reader
did or has to do.** A new `Warn` row in the capability table has to name a
difference the author can act on, and the table is allowed exactly one
(`internal/target/table_test.go`), so adding a second is a deliberate decision
and not a place to park a note to self.

## IR
Go structs are the schema source for their own surface: `internal/spec` derives the unresolved authoring schema, while `internal/ir` derives the resolved/debug schema. **Do not hand-author `.json` schema files.** Flow: `spec.Load` → `ir.Build` → `ir.Validate` → `generate.Generate`.

### Typed session state
A variable's `type:` is a single-line Python type expression, and `shapes:` is
a top-level **list** of named field groups it can refer to. A task's `assign:`
derives its finish fields from the destination variables, so authors declare
each type once. The grammar and error column live in
`internal/spec/typeexpr.go`; the vocabulary and advice live in
`internal/ir/shapes.go`. The shared emitted Python lives in
`internal/generate/shapes.go` and is inserted into both code targets verbatim.

No saved value enters a prompt automatically. A prompt grants access by naming
`{{variable}}` or `{{variable.field}}`. Omitted task and handoff history means
spoken `messages`; `reset` receives no old conversation. Task return restores
the owner's earlier context and adds only a completed or unserved status.

### No dictionaries in the authoring surface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slng-ai/unmute](https://github.com/slng-ai/unmute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
