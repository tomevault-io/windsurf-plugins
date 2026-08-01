---
trigger: always_on
description: Vector database. Go + C++ (internal/core/) + Rust (tantivy).
---

# Milvus

Vector database. Go + C++ (internal/core/) + Rust (tantivy).
pkg has its own go.mod (module: `github.com/milvus-io/milvus/pkg/v3`). Run `go get` from `pkg/` when adding dependencies there, not from root.

## Architecture

Coordinators manage metadata and scheduling; nodes execute work.
- Coordinators: rootcoord, datacoord, querycoordv2 (note the v2 suffix in directory names)
- Nodes: proxy (user-facing), querynodev2, datanode, streamingnode
- All component interfaces defined in `internal/types/types.go`

## Subsystems & Code Map

Each subsystem has a **top-level doc** (overview with links to sub-documents) and multiple **sub-documents** (detailed design, invariants, interfaces). The top-level doc alone is NOT sufficient — it is an index, not the content.

### Mandatory reading procedure

When your task modifies, explains, depends on, or affects a subsystem below, execute these steps IN ORDER before responding or writing code:

**Step 1 — Read the top-level doc.** Identify all sub-documents it links to.

**Step 2 — Read sub-documents.** The scope depends on task type:
- **Design tasks** (new feature, architecture change, cross-component change): Read **every** sub-document under the subsystem. No exceptions — design requires full-picture understanding. Do NOT judge relevance yourself; read all of them.
- **Targeted tasks** (bug fix, single-component change, code explanation): Read sub-documents that cover the components your task touches. When uncertain whether a sub-document is relevant, read it.

**Step 3 — Read source code** listed in each doc's "Key Packages" section. At minimum read the files directly related to your task.

**Step 4 — Cross-check** documentation against code. If they contradict, STOP and ask the user to resolve before proceeding.

NEVER answer based on documentation alone or code alone. NEVER skip Step 2 — this is the most common failure mode.

### Subsystems Reference

- [**Observability**](docs/agent_guides/observability/README.md): Logging, metrics, tracing, and observability debug workflows.
- [**Streaming System**](docs/agent_guides/streaming-system/streaming-system.md): Write path, WAL, DDL/DCL execution, replication && CDC.

## Testing

Go tests MUST use `-tags dynamic,test` and `-gcflags="all=-N -l"` (disable optimizations/inlining) or they won't compile / mockey-based monkey patching will fail:

```bash
go test -tags dynamic,test -gcflags="all=-N -l" -count=1 ./internal/querycoordv2/...
go test -tags dynamic,test -gcflags="all=-N -l" -count=1 ./internal/proxy/... -run TestXxx
```

Per-module shortcuts: `make test-querycoord`, `make test-proxy`, etc.

## Verification gate (MANDATORY before claiming "done" or pushing for review)

The reading procedure above tells you how to *enter* the code. This tells you how to *prove a change works*. A change is NOT verified by "it compiles + unit tests pass + success-path e2e is green." When the goal of a change is a **behavior** (retry / classification / routing / error-code propagation / fallback / cache invalidation / concurrency), execute these IN ORDER. Skipping them is the most expensive failure mode — it ships changes that are self-consistent but miss their purpose.

**G1 — Verify the data, not just your transform.** If your change adds a function/layer that maps or preserves a value X (an error code, a Status, a category, a flag), you only verified *your function*. Now verify its INPUT. Audit **every** place that constructs, throws, or rewrites X across the **whole repo** — not only the lines you edited. grep the escape hatches: `throw`, `.ToString()`/stringify, blanket fallbacks (catch-all → `Invalid` / `IOError` / `UnexpectedError`). A value destroyed or mis-set upstream makes your boundary logic dead code. Audit at the SOURCE of X, never only at the boundary that consumes it.

**G2 — Trace each real failure mode end-to-end.** Success-path e2e — even thousands of cases — does NOT exercise the failure modes a behavioral change exists for (S3 throttle, corrupt file, OOM, cancel, timeout, not-ready). For EACH one: either trace it by hand from origin → consumer, or fault-inject it, and confirm it lands in the intended bucket. "All green" on the happy path is not evidence the change works; it is only evidence you did not break the happy path.

**G3 — Do not over-claim.** Commit messages and PR body may assert ONLY benefits verified end-to-end via G1+G2. A benefit that depends on un-audited upstream or an un-triggered failure mode must be written as "follow-up" or "preserves codes for observability; retry wiring unverified" — never as achieved. A reviewer will verify your claim against the running system; over-claiming wastes their round.

**G4 — Adversarial self-review before human review.** Before pushing, do one pass asking: which failure mode have I NOT traced to its bucket? which upstream construction site of X have I NOT read? what would an adversarial reviewer grep for? Fix the gaps, or list them explicitly in the PR.

## Run Milvus Locally

```bash
scripts/start_standalone.sh    # start standalone mode
scripts/start_cluster.sh       # start cluster mode
scripts/stop_graceful.sh       # stop
scripts/standalone_embed.sh    # embedded standalone (no external deps)
```

## Code Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milvus-io/milvus](https://github.com/milvus-io/milvus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
