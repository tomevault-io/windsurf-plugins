---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository. It describes the active repository architecture.
Read `AGENT.md` for working practices and `SKILLS.md` for security-model rules.

## Current State

Syva v0.2 is the active product line: local `syva-core` + adapters + eBPF
enforcement. The v0.3 `syva-cp` control-plane experiment has been removed from
the active workspace. Historical CP notes are archived under `docs/archive/`.

Do not add new code that depends on `syva-cp`, `syva_control.proto`,
`cp_reconcile`, Postgres, node heartbeats, team ownership, or CP assignment
streams in this repository.

## Build And Test

Host-safe checks:

```bash
make macos-check
```

Linux full checks:

```bash
make fmt
make lint
make test
make precommit
make ci
```

Every `make` target wraps one source of truth: `cargo run -p xtask -- <cmd>`
(`fmt`, `lint`, `check`, `test`, `proto-check`, `check-release-docs`,
`check-ebpf-artifact-policy`, `eval-build`, `precommit`, `ci`, `build-ebpf`).
CI (`.github/workflows/ci.yml`) invokes the same xtask commands, so `make ci`
reproduces CI locally.

Run a focused test:

```bash
cargo test -p syva-core zone::tests::register_zone_is_idempotent  # one unit test
cargo test -p <crate> <name-substring>                           # by name
```

The eval suites live outside the workspace (own manifests, run via `--manifest-path`):

```bash
cargo test --manifest-path eval/oracle/Cargo.toml -- case_003 --exact --nocapture
cargo run  --manifest-path eval/harness/Cargo.toml               # spec harness
```

`build-ebpf` builds the release eBPF object by default because that is the
runtime artifact. Use `--debug` only for development.

macOS uses Lima for Linux verification:

```bash
make lima-up
make lima-check
make lima-shell
```

`make lima-check` runs format check, clippy, workspace check, workspace tests,
eval crate builds, and eBPF object compilation in the `syva-dev` Lima VM.

Privileged runtime evidence is separate (privileged Linux / BPF-LSM only; the
container gate also needs a container runtime). All are `#[ignore]`d in normal
`cargo test`:

```bash
sudo -E make verify-runtime              # load + attach 9 hooks + self-tests
sudo -E make verify-integration          # process/cgroup file_open denial (EPERM)
sudo -E make verify-container-integration # same denial against a real container
sudo -E make verify-audit-mode           # audit mode records would-deny without blocking
```

The gates above each start their own core. To verify a core that is
already running, use `make verify-deployment` (needs `SYVA_SOCKET`, default
`/run/syva/syva-core.sock`, plus a container runtime). The single-node Lima
deployment lifecycle is `make lima-bootstrap` → `lima-deploy` →
`lima-verify-deployment` → `lima-undeploy`, wrapped end to end by
`make lima-smoke`.

## Release-Doc Drift Guardrail

`cargo run -p xtask -- check-release-docs` (run by `make precommit`, `make ci`,
and the CI `guardrails` job) fails the build when tracked docs/code drift from
the v0.2 contract. It scans tracked `*.md`/`*.rs`/`*.proto`/`*.toml`/`*.yaml`
(excluding `docs/archive/`, and skipping fenced code blocks) and, outside code
fences, rejects:

```text
- stale hook counts: "7 hooks" / "seven hooks" / "8 hooks"  (v0.4 has nine)
- "syva_cgroup_attach"
- "cgroup_attach_task" UNLESS nearby text marks it a known gap
  ("not a bpf-lsm hook", "out of v0.2 scope", "do not reintroduce", assert!)
- "lima proves/verifies runtime|enforcement" without a "not"/"unless" caveat
- "debug ebpf" called the "default"/"runtime artifact" (release is the default)
```

It also requires `README.md`, `CLAUDE.md`, `AGENT.md`, `SKILLS.md`, and
`docs/release/v0.2-runtime-verification.md` to each mention all three runtime
gates: `verify-runtime`, `verify-integration`, `verify-container-integration`.
Preserve these invariants when editing docs (put illustrative trigger strings
inside code fences, which the checker skips). `xtask/src/main.rs` is exempt
because it stores the trigger strings as literals.

## Active Crates

| Crate | Binary | Purpose |
| --- | --- | --- |
| `syva-proto` | - | `syva.core.v1` protobuf API |
| `syva-core-client` | - | Unix-socket gRPC client for adapters |
| `syva-core` | `syva-core` | Linux eBPF enforcement engine and local API |
| `syvactl` | `syvactl` | Thin local operator CLI over `syva.core.v1` |
| `syva-adapter-file` | `syva-file` | TOML policy reconciler |
| `syva-adapter-k8s` | `syva-k8s` | `SyvaZonePolicy` CRD reconciler |
| `syva-adapter-api` | `syva-api` | REST proxy to local core |
| `syva-ebpf-common` | - | Shared userspace/eBPF C-layout types |
| `syva-ebpf` | - | Separate nightly eBPF workspace |
| `xtask` | - | Build/check helper |

Eval crates under `eval/` are outside the workspace and use their own
manifests.

API docs live under `docs/api/`. The protobuf/gRPC API (`syva.core.v1`) is the
source of truth; OpenAPI documents only the partial REST API; `syvactl` is a
thin local gRPC client. Keep `cargo run -p xtask -- check-api-docs` and
`cargo run -p xtask -- check-openapi` green when changing the control surface.

## Enforcement Model

`syva-core` populates BPF maps; nine eBPF LSM programs (`syva-ebpf/src/`) read

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [false-systems/syva](https://github.com/false-systems/syva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
