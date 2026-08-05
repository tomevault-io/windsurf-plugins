---
trigger: always_on
description: Operating manual for AI coding agents (Copilot CLI, GitHub Copilot,
---

# AGENTS.md

Operating manual for AI coding agents (Copilot CLI, GitHub Copilot,
Cursor, …) and human contributors working on **`vicondoa/d2b`
itself**. If you are *consuming* d2b in your own NixOS host
config, start at [README.md](./README.md) instead — this file is for
people changing the framework.

## What this is

d2b is an opinionated NixOS desktop microVM framework that
owns its microVM substrate end-to-end. The control plane is
**daemon-only**: `d2bd` supervises every per-VM DAG and
`d2b-priv-broker` dispatches every audited host mutation.
There are no per-VM systemd templates, no host-singleton framework
services, and no legacy bash CLI; see
[ADR 0015](./docs/adr/0015-daemon-only-clean-break.md) for the
binding architectural decision.

What the framework provides: per-env isolated networks with an
auto-declared NAT/DHCP "net VM", a per-VM `/nix/store` hardlink farm,
toggleable per-VM components (graphics, TPM, USBIP, audio), and the
versioned bundle/manifest contract that grounds the broker dispatcher.
See [README.md](./README.md) and
[`docs/explanation/design.md`](./docs/explanation/design.md) for the
full picture and threat model.

## Repo layout

```
.
├── README.md                       <- consumer-facing entry point
├── AGENTS.md                       <- this file
├── SECURITY.md                     <- disclosure policy + threat-model summary
├── CHANGELOG.md                    <- Keep a Changelog, grouped under `## [Unreleased]`
├── LICENSE                         <- Apache-2.0
├── flake.nix                       <- public surface: nixosModules / templates / checks
├── flake.lock
├── .github/workflows/              <- CI-only checks that stay out of root `flake.checks`
├── nixos-modules/                  <- THE framework
│   ├── default.nix                 <- aggregator imported as nixosModules.default
│   ├── options.nix / options-*.nix <- option schema (site / envs / vms)
│   ├── assertions.nix              <- eval-time invariants (CIDR overlap, platform gate, …)
│   ├── lib.nix                     <- internal helpers (subnetIp, mkMac, …)
│   ├── index.nix                   <- normalized internal VM/env/component index
│   ├── host.nix / host-*.nix       <- host activation, users, polkit, sidecars, keys, audit
│   ├── network.nix / net.nix       <- per-env bridges + auto-declared net VM
│   ├── store.nix                   <- per-VM /nix/store hardlink farm
│   ├── manifest.nix                <- JSON manifest emitter (versioned contract)
│   └── components/                 <- toggleable per-VM features
│       ├── graphics.nix            <- virtio-gpu + Wayland cross-domain
│       ├── tpm.nix                 <- per-VM swtpm 2.0
│       ├── usbip.nix               <- YubiKey USBIP passthrough
│       ├── home-manager.nix        <- HM-as-NixOS-module inside the guest
│       └── audio/{guest,host}.nix  <- vhost-user-sound + PipeWire mediation
├── pkgs/                           <- patched cloud-hypervisor / crosvm / vhost-device-sound
├── packages/                       <- Rust workspace; pinned rust-toolchain.toml
│   ├── d2b-core/              <- shared bundle DTOs, typed errors, privilege metadata
│   ├── d2b-host/              <- host-side lifecycle primitives (argv, hardlink farm, ifnames)
│   ├── d2b-contracts/          <- public + private wire contracts
│   ├── d2b/                   <- rust-native CLI
│   ├── d2bd/                  <- unprivileged public daemon / supervisor
│   ├── d2b-priv-broker/       <- privileged broker for audited host mutations
│   ├── d2b-guest-shell-runner/ <- standalone static guest helper for persistent shell feasibility
│   └── xtask/                     <- schema / docs codegen helpers; see
│                                      `docs/adr/0000` + `docs/adr/0009`
├── tests/                          <- see "Test layout" below
├── examples/                       <- minimal / graphics-workstation / multi-env / with-entra-id
├── templates/default/              <- `nix flake init -t github:vicondoa/d2b`
└── docs/                           <- Diataxis tree (explanation / how-to / reference)
                                       plus `docs/adr/` architecture decision records
```

New behaviour belongs in a focused file under `nixos-modules/`
(or `nixos-modules/components/` for per-VM toggles), wired in
from `nixos-modules/default.nix`. Don't fatten existing files.

## Build & validate

Use the top-level `Makefile` targets. The shell scripts under `tests/`
are implementation details unless a target or `tests/AGENTS.md` tells
you to run one directly.

```bash
# Sub-60s syntax + shellcheck loop for docs/shell-only edits.
make check-tier0

# Layer-1 local development umbrella: lint, Rust, proofs, flake,
# drift, and policy gates. CI runs these sub-targets in parallel.
make test-unit

# Focused Layer-1 shards when iterating on one surface.
make test-lint
make test-rust
make test-proofs
make test-flake
make test-drift
make test-policy

# PR-equivalent Layer-1 gate. Uses tests/layer1-jobs.json to run
# independent make test-* shards locally with bounded parallelism.
make check

# Legacy/full-static monolithic gate retained for explicit use.
make check-static

# Local Layer 1 + container integration. Still run the explicit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicondoa/d2b](https://github.com/vicondoa/d2b) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
