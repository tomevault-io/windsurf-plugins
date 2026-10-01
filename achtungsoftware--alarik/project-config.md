---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

Two deployables plus a shell-script test battery at the root:

- `alarik/` — the Swift (6.2, Vapor) server: S3 API, internal API, clustering. This is the SwiftPM package root.
- `console/` — Nuxt 4 SPA admin/object-browser console (`@nuxt/ui`), talks to the server's `/api/v1`.
- `*_tests.sh` at repo root — end-to-end suites that drive real server processes with `aws`, `rclone`, `mc`, and raw `curl`.
- `charts/alarik/` — Helm chart. `publish.sh` builds/pushes images and rewrites the version in `Sources/Global/Constants.swift`, `console/nuxt.config.ts`, and the chart's `appVersion` from `VERSION`.

## Commands

Swift package (run from `alarik/`):

```bash
swift build                                  # debug build -> .build/debug/Alarik
swift test --no-parallel                     # full unit suite (same as ./test.sh)
swift test --no-parallel --filter "S3ControllerTests"   # single suite
```

**Always pass `--no-parallel`.** Tests share on-disk state (`Storage/buckets`, `Storage/multipart`) process-wide, so parallel runs interfere. This holds for filtered runs too. CI (`.github/workflows/swift.yml`, macOS) runs exactly this.

Requires `isa-l` for the `CISAL` system library target: `brew install isa-l` (macOS) / `libisal-dev` (Debian).

End-to-end suites (run from repo root; each needs a `swift build` first and port 8080 free):

```bash
./run_all_tests.sh        # everything, ~20 min: unit + aws + rclone + mc + replication + cluster
bash aws_cli_tests.sh     # needs a server already running on :8080
./replication_tests.sh    # starts its own 2 instances (:8081/:8082)
./cluster_tests.sh        # starts its own 4 instances (:8091+), k=2/m=2
```

`run_all_tests.sh` starts each S3 suite against a freshly built binary in a temp state dir, so it never touches the repo's `Storage/`. Preflight requires `swift aws rclone mc jq`.

Running a server by hand:

```bash
cd alarik && JWT=test-secret ./.build/debug/Alarik serve --hostname 127.0.0.1 --port 8080
```

The process resolves `Storage/` relative to its cwd — run it from a scratch directory to get clean state.

Console: `cd console && npm run dev` (`:3000`). Whole stack in Docker: `./start_dev.sh` (server + console with hot reload), `./start_cluster.sh` (multi-node cluster compose).

### Debug vs release

Some bugs are release-only (optimizer-exposed; e.g. the ISA-L use-after-free in `ReedSolomonEngine`, #24). `swift test -c release` does not work here (the executable's `@main` collides with the test runner); reproduce release bugs by running a real multi-node cluster from `.build/release/Alarik`. Note that `#if DEBUG` seeds a fixed `alarik`/`alarik` admin and the `AKIAIOSFODNN7EXAMPLE` key — release seeds neither, so set `ADMIN_USERNAME`/`ADMIN_PASSWORD` and `DEFAULT_ACCESS_KEY`/`DEFAULT_SECRET_KEY`.

## Architecture

### No database — anywhere

There is no SQL/embedded DB. All control-plane metadata (users, buckets, access keys, policies, OIDC providers, outbox tasks, and cluster membership itself) lives in Alarik's own storage engine under the reserved pseudo-bucket `.alarik.sys` (`MetadataNamespace`), erasure-coded exactly like object data. `Storage/` is the only thing to back up. This is why rebalancing and bit-rot scrubbing cover metadata for free.

`MetadataStore` is the byte-oriented K/V layer over that: reads run locally from any node (gather-and-decode is idempotent), writes/CAS forward to the rank-0 owner so placements can't race. `MetadataListingService` fans out across nodes for listings; `localRecords` deliberately avoids fan-out for periodic sweeps (`O(nodes²)` upkeep traffic otherwise).

### Request path

`routes.swift` registers, in order: health probes (`/livez`, `/readyz`), `/api/v1/*` (console/internal API, mostly behind `InternalAuthenticator`), top-level node-to-node cluster controllers (behind the cluster-secret middleware), and finally `S3Controller`, which claims `:bucketName` + `**` catch-alls. Ordering matters — a bucket may not be named after a constant top-level path (`MetadataNamespace.reservedRootPaths`).

Every S3 object handler follows the same shape: authenticate → extract bucket/key → `ObjectRoutingService.routingDecision` (before touching `req.body`, since `.forward` streams it onward untouched) → serve locally or proxy to a responsible node. Multipart operations use `multipartRoutingDecision` instead, which pins to rank-0, because in-flight part state exists on only one node.

### Clustering

Opt-in: a node clusters only when both `CLUSTER_NODE_ADDRESS` and `CLUSTER_SECRET` are set; membership bootstraps from `CLUSTER_SEED_NODES`. Config is validated unconditionally at boot so typos fail fast.

- `PlacementService` — rendezvous (HRW) hashing, pure functions of `(bucket, key, activeNodes)`; unit-testable with no cluster. Top-3 is always a prefix of top-`k+m`, so rank-0 is primary in both views.
- `Services/ErasureCoding/*` — Reed-Solomon `k+m` via ISA-L (`CISAL`). Write/read/delete coordinators, rebalance on membership change, and a bit-rot scrubber. Defaults `k=4`/`m=2` (needs ≥6 nodes; `cluster_tests.sh` overrides to 2/2 for its 4 nodes).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achtungsoftware/alarik](https://github.com/achtungsoftware/alarik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
