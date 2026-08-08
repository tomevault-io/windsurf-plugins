---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**BearDrive** is the product name; **`bdrive`** is its CLI binary (file conventions: the `.bdrive/` settings directory and `.bdriveignore` at the project root, `~/.bdrive` home, `BDRIVE_HOME`). BearDrive is a Go CLI that mounts any folder as a synced volume: contents sync across devices and teammates through a **`bdrive serve` hub**, with accounts, per-file change history, and offline support. Clients are storage-blind — they sync through the hub over `https://` and never hold storage credentials; the hub owns the object store (S3, GCS, S3-compatible, or a plain directory) and devices converge through append-only journals in it. (Direct client-to-bucket sync without a hub is no longer supported; the object-storage backends exist only as the hub's own storage.)

The repo ships one binary: `cmd/bdrive` — the CLI, the sync daemon, and the web server (`bdrive serve`: viewer, uploads, multi-project sync hub).

## Commands

```sh
go build ./...                                   # build everything
go test ./...                                    # run all tests
go test ./internal/syncer -run TestConflict -v   # run a single test
go vet ./...                                     # vet
go build -o bdrive ./cmd/bdrive                  # build the binary (gitignored at repo root)

# web frontend (internal/webapp/frontend — only needed when changing frontend/src):
npm run build      # rebuild the committed assets in internal/webapp/static (run from frontend/)
npm run e2e        # Playwright suite against the seeded e2e hub (starts itself on :8993)
./check-dist.sh    # verify committed static/ matches frontend/src (run before releases)
```

There is no Makefile, linter config, or CI config in-repo. Releases run `goreleaser release` on a tagged commit (see `.goreleaser.yaml`); the version is injected via `-ldflags "-X main.version=..."` into `cmd/bdrive/main.go`. The frontend's built assets are **committed** (go:embed needs them in the module), so `go build` and `go install` never require Node — but a release tag must not ship a stale `internal/webapp/static`: run `frontend/check-dist.sh` first.

When testing the CLI manually, set `BDRIVE_HOME=/some/tmp/dir` to relocate all beardrive state (device identity, mount registry, volume stores) away from the real `~/.bdrive`.

## Architecture

Data flows in two hops; the local volume store is the pivot:

```
working folder  ←scan/materialize→  volume store (~/.bdrive/volumes/<vol>)  ←push/pull→  object store
 (real files)                       blobs/ + journal/ + state + sync                  s3:// gs:// file://
```

Package roles (`internal/`):

- **`journal`** — the core data model. Every change is an `Op` (`put`/`delete`) in a per-device append-only JSONL log. `Less` defines the total order `(lamport, time, device, seq)`; `Replay` folds all ops into the volume state, last-writer-wins per path. Everything else is machinery around this.
- **`store`** — a volume's local on-disk state: content-addressed blob store (`blobs/<aa>/<sha256>`), per-device journal copies, the per-mount materialization cache (`state-<mount-id>.json`, size+mtime fingerprints for cheap change detection), sync state (lamport clock + push cursor), and the exclusive flock that serializes cycles.
- **`remote`** — the `Backend` interface (Put/Get/List/Exists) with `file://`, `s3://`, `gs://`, and `https://` implementations (`https://` syncs through a `bdrive serve` server's `/api/store` API — the client device holds no storage credentials). `PutSigner` is the optional presign capability (S3/GCS). Remote layout: `blobs/<sha256>` + `journal/<device>.jsonl` under the URL prefix.
- **`syncer`** — the heart: `Session.Cycle()` runs one pass: scan → commit local ops → pull peer journals → preserve conflict copies → materialize merged state → push blobs + own journal. Read the package doc comment in `syncer.go` first. `ignore.go` holds the path filter (`.bdriveignore` rules + the `.bdrive` include list), applied symmetrically in scan and materialize; a newly filtered path is dropped from the cache *without* a delete op so opting out locally never deletes remotely.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runbear-io/beardrive](https://github.com/runbear-io/beardrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
