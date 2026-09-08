---
trigger: always_on
description: Guidance for AI coding agents (Cursor, Codex, Copilot, Gemini, Claude Code, …)
---

# AGENTS.md — civitai CLI

Guidance for AI coding agents (Cursor, Codex, Copilot, Gemini, Claude Code, …)
and human contributors. This is the **single source of truth** for stack,
conventions, the release process, and the few decisions that look wrong but are
intentional. For the contributor checklist see
[`CONTRIBUTING.md`](CONTRIBUTING.md).

🔴 **[`README.md`](README.md) IS THE PUBLISHED USER CONTRACT, AND IT IS NOT ONE
PLACE.** This file holds the decisions and rationale; the README states the
contract — exit codes, `--json` shapes, the command reference (don't re-derive
it here) — so **any** behaviour change is incomplete until it moves too, not
only the ones an item below cites. Its command section, exit-code table and
Troubleshooting index each state that contract and each goes stale ALONE: #371
shipped having updated two of three (#378). Grep every surface naming what you
changed: some of that prose is GENERATED (`internal/cmd/exitcodes_doc.go`) or
frozen verbatim by a provenanced fixture, so it cannot be fixed in place.

## Stack (exact versions — don't assume training-data defaults)

- **Go 1.25+** (`go.mod` says `go 1.25.0`). Single-binary CLI, no CGO
  (`CGO_ENABLED=0`).
- **Cobra v1.8.1** (command tree) + **Viper v1.19.0** (config), in the
  `gh` / `kubectl` / `stripe` mold.
- JSON Schema validation via **santhosh-tekuri/jsonschema/v6**; self-update via
  **minio/selfupdate**. Release tooling: **goreleaser v2**.
- **Interactive layer** (there IS one — this CLI is not print-only):
  **huh v1.0.0** (`app init`'s form), **bubbletea v1.3.10** + **bubbles v1.0.0**
  (`app dev-tunnel`'s spinner), **lipgloss v1.1.0** + **termenv v0.16.0** (every
  styled string). It all funnels through `internal/ui`; read its `CONVENTION.md`
  before adding color anywhere else.
- One job each: `x/term` (TTY detection, gating prompts + spinner),
  `x/crypto/ssh` (dev-tunnel), `x/net/html` (`internal/cmd/htmlrender.go`),
  `x/text/message` (required by jsonschema/v6's `LocalizedString`),
  `gopkg.in/yaml.v3` (config).
- Module path `github.com/civitai/cli`; the executable is `./cmd/civitai`.

## What this is

`civitai` is the Apps authoring CLI. `civitai app` scaffolds a correct
project, validates the manifest against the platform contract, packages it, and
submits it for review.

## Build / test / lint / run (exact commands)

```bash
make build   # -> bin/civitai, version ldflags from `git describe`
make test    # go test ./...
make vet     # go vet ./...
make fmt     # gofmt -s -w .
make lint    # golangci-lint; ERRORS with an install hint if it is not on PATH
make ci      # tidy + vet + test + build. NOT a mirror of CI — see below
```

🔴 **`make ci` DOES NOT RUN LINT, and this line used to claim it "mirrors GitHub
Actions CI".** It runs `tidy vet test build`; golangci-lint is a **separate CI
job**, version-pinned in `.github/workflows/ci.yml`. Not theoretical: a change
once reached a push with four fixtures emitting raw control bytes that only
staticcheck's ST1018 could see, because `make ci` was green and documented as
equivalent to CI. **Run `make lint` too before claiming done** — it errors when
golangci-lint is missing rather than degrading to something weaker, which is what
makes its zero meaningful; do not "helpfully" add a fallback.

CI (`.github/workflows/ci.yml`) runs **eight** jobs, not four steps:
`build-test` (vet + `gofmt -s -l .` + test + build), `lint`, `schema-drift`,
`pins-vs-published`, `ready-ack-runtime`, `template-page-vite`,
`template-page-money` and `scaffold-currency`.

🔴 **Reporting and gating are different questions, and fewer of those jobs gate
than run.** Item 11 carries the measured list of required contexts and the
instruction to re-measure before calling any job a gate — read it there; a second
copy here is how the original claim went stale. In particular **`lint` reports
but does not block a merge**, another reason to run it locally.

Run the binary you built with `./bin/civitai <cmd>`; per-package coverage is
`go test ./... -cover`.

## Shell & CI gotchas

These produce **clean exits and reassuring output while doing nothing** — the
expensive class. Read the tool's *output*, not just its exit code. Host-generic
shell traps were removed from here (they belong in your global rules); what
follows is specific to this repo's toolchain.

- **`gofmt -s -l .` checking zero files** prints nothing and exits 0 — same as
  "all clean". If a path is misquoted or the working tree is wrong, the clean
  verdict says nothing about the code. Verify the directory, and that the tool
  found files.
- **Build/test/tool not on PATH exits `127`; OOM exits `134`** — both non-zero,
  but a script reading `rc != 0` as "N errors found" reports a plausible wrong
  count. Prefer `make ci` (which handles this) over hand-rolled invocations, and
  assert a **minimum expected count** (≥1 package tested, ≥1 file checked) as a
  positive control.
- **`go test ./...` with a broken import in `_test.go`** can compile to 0 tests
  and pass. If a package you expect tests for is silent, check explicitly:
  `go test -v -count=1 ./path/to/pkg | head`.
- **`gh pr checks` / `gh pr view --json statusCheckRollup` pitfalls.** Kept

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [civitai/cli](https://github.com/civitai/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
