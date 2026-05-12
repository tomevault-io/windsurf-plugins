---
trigger: always_on
description: This file is the single source of truth for all AI coding agents (Claude Code,
---

# Project Instructions

This file is the single source of truth for all AI coding agents (Claude Code,
OpenAI Codex, or any tool that reads AGENTS.md / CLAUDE.md).

## Project Overview

Standalone SigmaStar video encoder with dual-backend architecture.

- Active implementation: `src/`
- Canonical documentation: `documentation/`
- Version tracking: `VERSION` (SemVer) + `HISTORY.md` (changelog)

## Development Workflow: spec - draft - simplify - verify

Every non-trivial task MUST follow this four-phase pipeline.
Each phase is a gate: do not advance until the current phase passes.

### Phase 1: Spec (Plan)

Before writing any code, produce a plan:

1. Read relevant documentation in `documentation/`.
2. Read the source files you intend to modify.
3. Write a concise plan covering: what changes, which files, why.
4. Document key design decisions and their rationale in the plan. This
   prevents oscillating between approaches mid-implementation.
5. Get human approval on the plan before proceeding.

Do NOT skip planning. A good plan lets you one-shot the implementation.

### Phase 2: Draft (Implement)

Execute the plan:

- Follow the coding conventions below.
- Make minimal, focused changes. Do not refactor unrelated code.
- Do not add features beyond what the spec calls for.
- **Verify incrementally**: run `make lint` after each logical change (new
  function, changed struct, modified pipeline stage). Do not batch all
  verification to Phase 4. Catching errors early prevents compounding
  failures that are harder to diagnose.
- If a change touches both backends, build each backend separately after
  the change: `make build SOC_BUILD=star6e`, then
  `make build SOC_BUILD=maruko`. Both output to separate directories
  (`out/star6e/`, `out/maruko/`) — no clean needed between them.

### Phase 3: Simplify (Review)

After implementation, review your own work:

- Can any function be shorter or clearer?
- Are there unnecessary abstractions, error paths, or comments?
- Does the architecture stay clean? No dead code, no orphan headers.
- Remove anything that is not strictly needed.

### Phase 4: Verify (Build + Test)

Run verification before declaring done:

```
make verify
```

This builds both backends and checks that all expected binaries exist.
If verify fails, follow the **Error Recovery Loop** below. Do not skip this step.

When connected devices are available, run targeted deployment tests after
`make verify` passes. Only rows with a **Host** filled in under
**Operational Defaults → Deployment Targets** are tested; blank rows are
skipped automatically.

For `venc` on the current Star6E bench (`root@192.168.1.13` / imx335), prefer
the direct JSON-config helper first:

```bash
scripts/star6e_direct_deploy.sh cycle
```

This validates the production `/etc/venc.json` path, daemon startup, HTTP API,
and `/tmp/venc.log` capture.

Use `make remote-test` for bounded CLI validation:

- sensor capability discovery (`--list-sensor-modes`)
- max-FPS sweeps across reported modes
- Maruko runtime deployment runs that still depend on `/tmp` staging

For each host row that uses `remote-test`, first probe sensor capabilities
with `--list-sensor-modes`, then test **every listed sensor mode** at its
reported max FPS. This catches per-mode regressions and cold-boot unlock
failures.

```
# 1) List sensor modes
make remote-test ARGS='--host root@<HOST> --soc-build <backend> --run-bin venc -- --list-sensor-modes --sensor-index <idx> [--isp-bin <path>]'

# 2) Test each mode at its max FPS (repeat for every mode reported above)
make remote-test ARGS='--host root@<HOST> --soc-build <backend> --run-bin venc -- --sensor-index <idx> --sensor-mode <M> -f <MAX_FPS> [--isp-bin <path>]'
```

**Example** — Star6E / imx335 at `192.168.1.13`, where `--list-sensor-modes`
reports modes 0 (30fps), 1 (60fps), 2 (90fps), and 3 (120fps):

```
make remote-test ARGS='--host root@192.168.1.13 --soc-build star6e --run-bin venc -- --list-sensor-modes --sensor-index 0'
make remote-test ARGS='--host root@192.168.1.13 --soc-build star6e --run-bin venc -- --sensor-index 0 --sensor-mode 0 -f 30'
make remote-test ARGS='--host root@192.168.1.13 --soc-build star6e --run-bin venc -- --sensor-index 0 --sensor-mode 1 -f 60'
make remote-test ARGS='--host root@192.168.1.13 --soc-build star6e --run-bin venc -- --sensor-index 0 --sensor-mode 2 -f 90'
make remote-test ARGS='--host root@192.168.1.13 --soc-build star6e --run-bin venc -- --sensor-index 0 --sensor-mode 3 -f 120'
```

Substitute `<HOST>`, `<backend>`, `<idx>`, and `--isp-bin` from the
Deployment Targets table. The mode indices and FPS values are examples for
imx415; always use the actual values reported by `--list-sensor-modes`.

If any mode's max-FPS run fails, check whether the sensor unlock sequence
fired (see `documentation/SENSOR_UNLOCK_IMX415_IMX335.md`).

Do not block a PR solely because a device is offline — only rows with a host
are tested.

For a full pre-PR check (version bump, changelog, build):

```
make pre-pr
```

## Remote-Test Interpretation

After `make verify` passes and devices are available, deployment tests are the
primary feedback mechanism for validating changes on real hardware.
The `remote_test.sh` workflow emits strict exit codes and an optional JSON

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenIPC/waybeam_venc](https://github.com/OpenIPC/waybeam_venc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
