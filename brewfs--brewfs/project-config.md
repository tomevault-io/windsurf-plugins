---
trigger: always_on
description: This repository is a Rust/FUSE distributed filesystem. Treat correctness,
---

# BrewFS Agent Guide

This repository is a Rust/FUSE distributed filesystem. Treat correctness,
POSIX behavior, and repeatable performance evidence as first-class
requirements. Do not accept a performance change because a single focused
benchmark improves.

## Repository Map

- `src/vfs/`: FUSE-facing VFS, read/write path, writeback cache, stats, and
  memory budgeting.
- `src/chunk/`: chunk layout, object reads/writes, cache, compression, and
  compaction helpers.
- `src/meta/`: metadata client and Redis/SQLx/etcd/TiKV stores.
- `src/cadapter/`: object backend adapters, including S3-compatible storage.
- `docker/compose-xfstests/`: BrewFS and JuiceFS perf runners.
- `docker/compose-pjdfstest/`: POSIX smoke tests.
- `tools/perf/`: profiling/flamegraph helpers. Use these for diagnosis, not as
  the sole acceptance baseline.
- `doc/testing/bench.md`: local benchmark and profiling guidance.
- `docker/compose-xfstests/aliyun/README.md`: current managed-backend
  performance methodology and parity rules.
- `doc/superpowers/plans/`: detailed implementation and experiment logs.

## Development Discipline

- Read the relevant module and nearby tests before editing.
- Keep changes small and tied to one hypothesis.
- Use `rg`/`rg --files` for search.
- Use `apply_patch` for manual edits.
- Do not revert user changes or unrelated worktree changes.
- Do not leave failed performance code in the tree. Revert rejected candidates
  with a targeted patch, then record the artifact and reason.
- For Rust behavior changes, prefer TDD: write or identify a failing test,
  prove it fails, then make the smallest implementation change.

## Local CI Gate For Accepted Code

Every accepted code change, including performance work, must pass the relevant
local CI gate before commit. At minimum, run the Rust job commands from
`.github/workflows/ci.yml`:

The active performance goal treats the workflow's `Test workspace` step as a
hard validity gate for every accepted optimization attempt. Run
`CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo test --workspace --lib --bins`
locally in the same iteration before accepting perf numbers, updating the
README comparison table, or committing code. Focused unit tests are useful
during development, but they do not replace this CI test gate.

```bash
cargo fmt --all --check

bash -n docker/compose-xfstests/run_perf_in_container.sh
bash -n docker/compose-xfstests/run_redis_perf.sh
bash -n docker/compose-xfstests/run_juicefs_perf_in_container.sh
bash -n docker/compose-xfstests/run_juicefs_perf.sh
bash docker/compose-xfstests/test_perf_report_delta.sh
bash docker/compose-xfstests/test_juicefs_direct_matrix.sh
bash docker/compose-xfstests/test_juicefs_perf_report.sh

CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo check --workspace
CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo build --workspace

metadata="$(cargo metadata --no-deps --format-version 1)"
if python3 -c 'import json, sys; data = json.loads(sys.stdin.read()); members = set(data["workspace_members"]); sys.exit(not any(pkg["name"] == "rfuse3" and pkg["id"] in members for pkg in data["packages"]))' <<< "$metadata"; then
  cargo check -p rfuse3 --no-default-features --features tokio-runtime
  cargo check -p rfuse3 --no-default-features --features io-uring-runtime
  cargo check -p rfuse3 --no-default-features --features async-io-runtime
fi

CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo check -p brewfs --no-default-features --features fuse-tokio-runtime
CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo check -p brewfs --no-default-features --features fuse-io-uring-runtime
CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo test --workspace --lib --bins
CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0 cargo clippy --workspace
git diff --check
```

If disk space is tight, remove rebuildable Rust target directories before perf
runs. Do not delete accepted perf artifacts unless explicitly asked.

## Performance Acceptance Rules

Use compose runners as the acceptance baseline. Use `tools/perf/` only after a
compose gap identifies where profiling is needed.

Focused writeback comparison command:

```bash
PERF_LOG_TO_CONSOLE=false PERF_FIO_SIZE=512m PERF_FIO_RUNTIME=20 \
  bash docker/compose-xfstests/run_redis_perf.sh --s3 \
  --writeback-throughput-profile \
  --tools "fio-seqwrite fio-randwrite fio-randrw"
```

JuiceFS focused comparison command:

```bash
PERF_LOG_TO_CONSOLE=false PERF_FIO_SIZE=512m PERF_FIO_RUNTIME=20 \
  bash docker/compose-xfstests/run_juicefs_perf.sh \
  --writeback-throughput-profile \
  --tools "fio-seqwrite fio-randwrite fio-randrw"
```

Direct-IO guard for writeback-sensitive changes:

```bash
PERF_LOG_TO_CONSOLE=false PERF_FIO_DIRECT=1 PERF_FIO_SIZE=512m PERF_FIO_RUNTIME=20 \
  bash docker/compose-xfstests/run_redis_perf.sh --s3 \
  --writeback-throughput-profile \
  --tools "fio-seqwrite fio-randwrite fio-randrw"
```

For a broad acceptance pass, include all core fio scenes and metadata stress:

```bash
PERF_LOG_TO_CONSOLE=false PERF_FIO_SIZE=512m PERF_FIO_RUNTIME=20 \
  bash docker/compose-xfstests/run_redis_perf.sh --s3 \
  --writeback-throughput-profile \
  --tools "fio-seqread fio-seqwrite fio-randread fio-randwrite fio-randrw fio-bigread fio-bigwrite metaperf dirstress dirperf"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brewfs/brewfs](https://github.com/brewfs/brewfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
