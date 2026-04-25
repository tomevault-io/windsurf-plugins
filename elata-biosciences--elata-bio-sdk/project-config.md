---
trigger: always_on
description: This file is for AI coding agents working in this repository. Use it as a
---

# AI Agent Guide

This file is for AI coding agents working in this repository. Use it as a
practical playbook for understanding the repo, choosing the right workflow, and
avoiding common mistakes.

## What This Repo Is

Elata SDK is a mixed Rust + TypeScript monorepo for biosignal tooling:

- EEG core crates and WASM bindings
- Web Bluetooth EEG headset transport (`eeg-web-ble`; Muse built-in, extensible)
- rPPG processing for web
- Native FFI layers for mobile/native clients
- Demo scaffolding via `create-elata-demo`

The repo is not a generic JS monorepo. Many changes cross Rust, generated WASM,
TypeScript wrappers, demo apps, and release tooling.

## First Things To Read

When starting work, orient with these files first:

- [README.md](README.md): repo overview, package list, build/demo commands
- [run.sh](run.sh): canonical task runner for build, test, release, and local package workflows
- [CONTRIBUTING.md](CONTRIBUTING.md): contribution and verification expectations
- [docs/guides/ai-assisted-development.md](docs/guides/ai-assisted-development.md): map for AI agents—`docs/` vs `elata-docs/` tutorials vs package `README`/`llms.txt` (includes vendor headset paths)
- [docs/releasing.md](docs/releasing.md): release flow and publish rules
- [docs/create-elata-demo.md](docs/create-elata-demo.md): canonical scaffolding workflow

For package-specific work, read the nearest package README and `package.json`
before changing code.

Treat `docs/implementation-plan-*.md` as planning or historical context unless
they clearly match the current code. For operational truth, prefer `run.sh`,
package `package.json` scripts, package READMEs, and maintainer/scaffolding
docs.

## Repo Map

- `crates/`: Rust crates for EEG, rPPG, protocol support, FFI, and bridges
- `packages/eeg-web`: TS wrapper around generated EEG WASM bindings
- `packages/eeg-web-ble`: Web Bluetooth transport for EEG headbands — `src/transport/` (`BleTransport`) vs `src/devices/muse/` (Muse protocol); open to additional `src/devices/` modules
- `packages/rppg-web`: TS wrapper and demo tooling for the rPPG pipeline
- `packages/create-elata-demo`: published scaffolder for demo apps
- `eeg-demo/`: in-repo EEG browser demo
- `ios-demo/`, `android-demo/`: native demos
- `scripts/`: helper scripts used by package and release flows
- `docs/`: architecture, scaffolding, and release docs

## Canonical Commands

Prefer these repo-level commands over ad hoc package commands when possible:

- `./run.sh doctor`: fast health check for toolchain, repo state, and artifacts
- `./run.sh dev [eeg|rppg|all]`: build debug artifacts
- `./run.sh build [eeg|rppg|all]`: build release artifacts
- `./run.sh demo [eeg|rppg|hal]`: run demo flows
- `./run.sh test`: run Rust and web test suites
- `./run.sh test create-elata-demo`: run scaffolder tests plus template smoke builds
- `./run.sh verify-all`: run publish-grade verification
- `./run.sh changeset`: create a changeset for releasable work

If a package README and `run.sh` disagree, inspect `run.sh` and current
`package.json` scripts before deciding the package README is authoritative.

## Current Source Of Truths

These are easy places to get confused:

- `create-elata-demo` is the preferred scaffolding path for new demo apps.
- for browser rPPG integration, `createRppgSession()` is the preferred app entrypoint
- `sync-to` still exists, but it is an internal EEG local-dev helper.
- `sync-to` only builds and links `packages/eeg-web`; it is not a general repo sync command.
- `scripts/dev-link.sh` is only a backward-compatible wrapper around `run.sh sync-to`.
- `pnpm` is the preferred repo package manager, but workspace behavior matters.

## Wrong-Path Prevention

When writing docs, answering questions, or generating examples, reduce the
chance that consumers follow an internal or legacy-looking path:

- Lead with the canonical consumer path first:
  - new app or evaluation: `@elata-biosciences/create-elata-demo`
  - existing browser app: published packages such as `@elata-biosciences/eeg-web`, `@elata-biosciences/eeg-web-ble`, or `@elata-biosciences/rppg-web`
- Explicitly label internal workflows as internal when they appear:
  - `./run.sh sync-to`
  - `scripts/dev-link.sh`
  - in-repo demos used for SDK development
  - historical `docs/implementation-plan-*.md` files
- Do not present internal helpers and consumer onboarding flows as equivalent options.
- If mentioning a non-default path, explain who it is for, why it exists, and why the default path is still preferred.
- If a user asks "which path should I take?", answer in this shape:
  - recommended default
  - only use the alternative when a specific repo-maintainer or advanced-integration condition applies
- For browser rPPG work, start with `createRppgSession()` and only drop to generated WASM bindings if you are intentionally debugging the SDK itself.
- If a reported consumer issue might actually be workspace coupling, check the `pnpm --ignore-workspace` caveat before concluding that the scaffold or template is broken.

## Important Gotcha: Scaffolding Inside This Repo

If a scaffolded app is created inside this repository, `pnpm install` from that
app directory may still bind to the parent workspace defined in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elata-Biosciences/elata-bio-sdk](https://github.com/Elata-Biosciences/elata-bio-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
