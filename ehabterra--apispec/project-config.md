---
trigger: always_on
description: Guidance for AI coding agents (and new contributors) working on **apispec** —
---

# CLAUDE.md

Guidance for AI coding agents (and new contributors) working on **apispec** —
a static-analysis tool that reads Go source and generates OpenAPI 3.1 specs.

## Architecture in one pass

Pipeline (each stage consumes the previous one's output):

1. **`internal/metadata`** — loads packages (AST + go/types) and records
   *facts*: functions, types, assignments, call-graph edges, string-pooled.
   No spec decisions happen here.
2. **`internal/spec` tracker** (`tracker.go`, `lazy_tree.go`) — expands the
   call graph from route-registration sites into a tracker tree
   (LazyTree is the default engine).
3. **`internal/spec` extractor** (`extractor.go`, `pattern_matchers.go`) —
   walks the tree with **config-driven patterns** (per-framework
   `config_<framework>.go`) to find routes, request/response bodies, params,
   security.
4. **`internal/spec` mapper** (`mapper.go`, `schema_mapper.go`) — resolves Go
   types to OpenAPI schemas/components.
5. **`internal/engine`** — orchestrates 1–4; **`internal/core`** detects the
   framework; **`generator/`** is the public library API; **`cmd/apispec`**
   the CLI, **`cmd/apispecui`** the web UI.

Design docs live in `docs/` — most importantly `TYPE_MODEL.md` (structured
type model + migration phases), `TRACKER_REDESIGN.md`,
`AUTH_DETECTION_DESIGN.md`, `INTERFACE_RESOLUTION.md`.

## Commands

```bash
make test            # go test ./...
make lint            # golangci-lint (CI runs this + vet + gofmt)
make fmt
make coverage
go test ./internal/spec -run TestName -v          # one test
go test ./internal/metadata -run TestGenerateMetadata -update
                     # rewrite the metadata golden files (internal/spec/tests/*.yaml)
scripts/compare-spec.sh                            # regenerate/diff fixture snapshots
```

## Profiling & performance

```bash
./apispec --dir <project> --cpu-profile --mem-profile --profile-dir profiles
                     # also: --block-profile --mutex-profile --trace-profile
go tool pprof -top ./apispec profiles/cpu.prof
go tool pprof -list 'FuncName' -sample_index=alloc_space ./apispec profiles/mem.prof
make metrics-generate     # per-stage metrics JSON (--custom-metrics --metrics-path)
make metrics-view         # interactive metrics viewer (scripts/view_metrics.sh)
```

- First-line diagnosis is the per-stage `[engine]` log lines (loaded /
  metadata generated / tracker tree / spec mapped). With LazyTree, tree
  expansion happens *inside* the "spec mapped" stage — a bigger mapping
  number alone is moved work, not necessarily a regression.
- Benchmark on a large real project, never on `testdata/` fixtures — they
  are so small that `go/packages` load noise dominates. A/B by building
  binaries from two `git worktree`s of the versions under comparison.
- The extraction walk visits every tracker node: any per-child O(depth)
  work (string concatenation, capped-cap slice appends) goes quadratic over
  deep graphs and shows up as GC dominance (`runtime.scanObject`,
  `runtime.madvise`) rather than as the guilty frame. Chain identity is
  interned to int handles (`chainInterner`, extractor.go) for exactly this
  reason — extend the interner rather than reintroducing key strings.

## Golden rules (hard-won invariants — do not relearn these)

1. **Determinism is a feature.** Any map iteration whose order can reach the
   output (spec, metadata YAML, component names, operationIds) must be
   sorted. This was the root cause of a long-standing flaky-output bug;
   `TestGenerateMetadataDeterministic` / `TestGenerateDeterministic` guard it.
2. **Never parse type strings.** Build or accept a `typemodel.TypeRef`
   (`internal/typemodel`); pooled type strings parse once via the memoized
   `Metadata.TypeRefOf` / `CallArgument.TypeRef()` (shared refs — `Clone()`
   before mutating); render to a string only at an output boundary. The
   transitional string views are deleted — do not reintroduce them. See
   `docs/TYPE_MODEL.md`.
3. **Metadata type-name conventions.** The `Types` map keys a declaration by
   its bare name (`"Page"`, parameters in `Type.TypeParams`); the bracketed
   `"Page[T]"` form exists only as the methods-table key (matching how a
   generic receiver renders). Two string-surgery islands are deliberate
   naming behavior, not parsing debt: the mapper's map branch and the
   argument renderer's qualification tail (see `docs/TYPE_MODEL.md`).
4. **Layering: metadata records facts, spec decides schemas.** External-type
   registry, config overrides, and marshaler-based decisions belong in the
   spec layer, never at metadata time (collapsing types early loses formats —
   e.g. the historical `uuid.UUID` regression). Inline (non-`$ref`) types
   must skip every `$ref` fast-path or refs dangle.
5. **Detection must be framework-agnostic and config-driven.** Any new
   detection capability (auth, params, bodies) has to work for all supported
   frameworks (gin, echo, chi, fiber, gorilla/mux, net/http) and all wiring
   styles (router-level, group, per-route, wrapper, var-assigned) via the
   pattern configs — never hardcoded for the framework that prompted it.
6. **Bound tracker work by cumulative cost, not depth.** Tree expansion over

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehabterra/apispec](https://github.com/ehabterra/apispec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
