---
trigger: always_on
description: > **Prime directive:** If I wanted hacks, I'd write it myself. **Don't ever choose hacky over correct.**
---

# AGENTS.md — Guide for AI Agents Working on OpenSysML

> **Prime directive:** If I wanted hacks, I'd write it myself. **Don't ever choose hacky over correct.**
> Fix root causes upstream, not symptoms. Never weaken, skip, or delete tests to make them pass.

OpenSysML is a production-grade SysML v2 implementation in **Go 1.23+** (module `github.com/Open-MBEE/OpenSysML`).
It provides a hand-written lexer/parser, semantic engine, execution runtime, LSP server (`sysml-lsp`), and REPL (`sysml`).

---

## 1. Golden Rules

1. **Correctness over expedience.** No shortcuts, no stubs left behind, no lossy conversions. If a proper fix is large, do it properly or stop and flag it.
   - **For features specifically: do not minimize code changes or dodge complexity.** Implement the feature fully and correctly even if it touches many files, adds new types, or requires refactoring. Completeness beats diff size. See §8.
2. **Root-cause first.** Before editing, confirm *why* something fails (read the code, add a temporary debug print, write a focused test). Then make the minimal correct change.
3. **Never regress.** `develop` is green. Any test passing on `develop` must still pass on your branch. Diff against `develop` if unsure: `git stash && git checkout develop && go test ./... ; git checkout - && git stash pop`.
4. **Respect the architecture invariants** (see §4). The AST is immutable; semantics live in side tables; execution consumes lowered IR — do not bypass these.
5. **Tests are the contract.** Existing tests encode intended behavior (including *when* and *where* errors surface). Make code satisfy tests, not the reverse — unless the test is provably wrong, in which case explain before changing it.
6. **Leave no dead code.** Remove superseded helpers/structs. Run `go vet ./...` to catch it.

---

## 2. Build, Test & Verify

Use the Makefile (preferred) or raw `go` commands. **Never `cd`** — run from repo root.

```bash
make build            # build bin/sysml and bin/sysml-lsp (with version ldflags)
make build-sysml      # REPL binary only
make build-lsp        # LSP binary only
make test             # full suite: go test -race -coverprofile ... ./...
make test-shard SHARD=runtime   # one CI shard of the race suite (runtime|model|export|rest)
make test-short       # faster, no race detector
make clean            # remove build artifacts
```

Raw equivalents / targeted runs:

```bash
go build ./...                                   # must always be clean
go vet ./...                                     # must be clean (catches unused/dead code)
go test ./...                                    # all tests
go test ./internal/exec/runtime/...             # one package tree
go test -run TestExecutionConformance ./internal/exec/runtime
go test -race ./...                             # race detector (CI runs this)
gofmt -l .                                      # must print nothing (CI enforces gofmt)
```

**Definition of done for any change:** `go build ./...`, `go vet ./...`, `gofmt -l .` (empty), and `go test ./...` all pass. Paste the results.

The OMG training-corpus gate is part of that suite but skips while the corpus is absent, so
fetch it once with `./scripts/download-training-examples.sh` and re-run
`go test -count=1 ./tests/corpus -run TestTrainingExamples`. CI downloads the corpus
too and sets `OPENSYSML_REQUIRE_TRAINING_CORPUS=1`, so there an absent corpus fails rather
than skips.

The three OMG pilot corpora are gated the same way: fetch them with
`./scripts/download-pilot-corpora.sh` and run
`go test -count=1 ./tests/corpus -run TestPilotCorpora`. CI sets
`OPENSYSML_REQUIRE_PILOT_CORPORA=1`. See `docs/project/pilot-corpora.md`.

So is the pilot's XMI of the standard library, which the identity gate reads: fetch it with
`./scripts/download-pilot-library-xmi.sh` and run
`go test -count=1 ./tests/identity -run TestPilotLibraryXMI`. CI sets
`OPENSYSML_REQUIRE_PILOT_LIBRARY_XMI=1`. So is the OMG PSSM test suite, which the SysML v1
migrator is gated over: fetch it with `./scripts/download-pssm-suite.sh` and run
`go test -count=1 ./tests/corpus -run TestPSSMSuiteMigration`. CI sets
`OPENSYSML_REQUIRE_PSSM_SUITE=1`. See `docs/project/pssm-migration.md`. Whatever sets a require
variable must run the matching download script first; the scripts are idempotent, and none
reports success over an empty corpus.

The Modelica Reference-FMUs gate the FMI integration the same way: fetch them with
`./scripts/download-reference-fmus.sh` (into `examples/reference-fmus/`, gitignored), install
the runner's simulator with `pip install fmpy`, and run
`go test -count=1 ./tests/fmi -run TestReferenceFMUs`. CI sets
`OPENSYSML_REQUIRE_REFERENCE_FMUS=1`.

All four roots share one mechanism (`tests/corpus/corpus_gate_test.go`) but two
policies, and the difference is deliberate: the training corpus is **asserted** clean, so its
expectation file holds no per-file counts and `-update-training` refuses to record one, while
the other three are a **per-file ratchet** whose every movement must be adjudicated. Do not
turn the assertion into a ratchet.

The RDF mapping has a per-file ratchet of its own over every model under `examples/`, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-MBEE/OpenSysML](https://github.com/Open-MBEE/OpenSysML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
