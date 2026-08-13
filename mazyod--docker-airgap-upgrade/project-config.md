---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Six standalone Bash scripts for upgrading Docker Engine 29.1.5 → 29.6.2 (and containerd.io 2.2.1 → 2.2.6) on **air-gapped RHEL 8/9 servers**. There is no application code, no build system, no package manager, and no test framework — the deliverable is the scripts themselves, bundled into a tarball and hand-carried to disconnected servers.

The scripts run as root on RHEL. They cannot be executed on the macOS dev machine.

## Working on this repo

There is no lint/test tooling configured. The practical checks:

```bash
bash -n upgrade-docker.sh                 # syntax check (works on macOS)
shellcheck upgrade-docker.sh              # available via homebrew
```

All six scripts are currently `bash -n` clean and `shellcheck` clean. Keep them that way — where a suppression is genuinely warranted, use an inline `# shellcheck disable=SCxxxx` with a reason, not a blanket ignore.

**Real execution happens through `tests/vm/`** — an OrbStack-based harness that runs the actual scripts against Rocky Linux 9 (x86_64, systemd PID 1) on the dev Mac:

```bash
tests/vm/bootstrap-vm.sh      # build the S1 baseline
tests/vm/build-bundle.sh      # real download-docker-packages.sh run
tests/vm/tier2-run.sh         # Tier 2 cases: reject / upgrade / rollback
tests/vm/negative-control.sh  # prove test 2.4 catches the regression
tests/vm/reset-baseline.sh    # back to S1 between destructive runs
```

The S1 baseline deliberately puts containerd's root on a **separate XFS filesystem at `/data/containerd`** with real images, containers and volume data on it. That is the configuration the pre-v2.0.0 phase 6 destroyed, and it is the only way to test it. See `tests/vm/README.md` for what this proves and — importantly — what it does not (no Swarm, not real RHEL, no GPU, not bare metal).

`simulate-upgrade.sh` remains a separate dnf-path smoke test. It is **not** the same code path as `upgrade-docker.sh` (see below), so passing it does not prove the air-gapped path works.

## Two different install strategies (important)

The repo deliberately contains two incompatible ways of installing the same packages:

| Script | Strategy | Why |
|--------|----------|-----|
| `simulate-upgrade.sh` | `createrepo` + `dnf install` then `dnf distro-sync --allowerasing` | Two-phase dnf; plain `dnf upgrade` is a no-op here |
| `upgrade-docker.sh` | `rpm -Uvh --force` on a validated file list | Corporate satellite servers break dnf with `SSL certificate problem: EE certificate key too weak` |

`recover-dnf.sh` prints recovery commands that reference an `--enablerepo=docker-local` repo. That repo only exists on machines that ran the simulation path; on production air-gapped hosts it must be created first or the commands will fail.

## Current upgrade scope (read this before changing phase logic)

The cluster is on 29.1.5 / containerd.io 2.2.1. This upgrade stays **inside the containerd 2.2.x line**. That is why several things the previous round required are gone:

- **No containerd config migration or regeneration.** 2.2.1 and 2.2.6 share config v3. Regenerating would discard a relocated `root` path, registry mirrors and runtime config — silently repointing a node at an empty `/var/lib/containerd`. Phase 6 verifies; it does not rewrite.
- **No XFS `ftype=1` check.** Any node running containerd 2.x has already satisfied it.
- **No automatic orphaned-network cleanup.** Extracted to `clean-swarm-networks.sh`, run on demand.

All three are recoverable from git history at `upgrade-docker.sh` v1.2.3 (commit `974683a`) if a future containerd **major** upgrade needs them back. Do not resurrect them for a point release.

**The whole-cluster-together rule does not apply here.** 29.1.5 and 29.6.2 both speak containerd 2.2.x gRPC, so a mixed 29.1.5/29.6.2 Swarm is fine and nodes roll one at a time. The rule still holds across the 1.7 ↔ 2.x boundary.

## Invariants the scripts depend on

- **Service order is not optional.** Stop `docker` → `docker.socket` → `containerd`. Start `containerd` → wait for readiness → `docker`. Readiness means polling `ctr version` (up to 30×2s) **and** `ctr snapshots --snapshotter overlayfs ls`, because systemd reports containerd active before its snapshotter is usable. A bare `sleep` is not readiness.
- **Verify the stop, don't assume it — and fail closed.** After stopping, confirm `docker`, `docker.socket` and `containerd` are all conclusively stopped before touching packages or network state. Use the `verify_unit_stopped()` helper, not `systemctl is-active`: `is-active` returns nonzero for `activating`, `deactivating`, *and* for failing to reach systemd, so treating nonzero as "stopped" fails open. The helper requires a successful `systemctl show`, an ActiveState of `inactive` or `failed`, and `MainPID=0`. `docker.socket` specifically: if it survives while dockerd is down, anything touching the socket socket-activates dockerd again, mid-transaction.
- **The package is `containerd.io`, never `containerd`.** The standalone `containerd` package is a different, wrong thing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mazyod/docker-airgap-upgrade](https://github.com/Mazyod/docker-airgap-upgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
