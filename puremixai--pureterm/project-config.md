---
trigger: always_on
description: PureTerm is an open-source SSH/SFTP client built with Cordis, ssh2, and xterm.js. It has two local entry points: Electron Desktop and standalone local Web. SSH is always initiated by the user’s computer; local Web binds only to loopback and provides no public service, user accounts, tenant isolation, or remote-control tunnel.
---

# AGENTS.md

[中文版本](AGENTS_zh.md)

PureTerm is an open-source SSH/SFTP client built with Cordis, ssh2, and xterm.js. It has two local entry points: Electron Desktop and standalone local Web. SSH is always initiated by the user’s computer; local Web binds only to loopback and provides no public service, user accounts, tenant isolation, or remote-control tunnel.

These instructions apply to the whole repository. Before changing `packages/`, `apps/`, or root scripts, read the [architecture](docs/architecture.md) and [layout decision](LAYOUT-PROPOSAL.md). Before changing release behavior, read the [Desktop release guide](docs/desktop-release.md). The [deepseek-harness AGENTS.md](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md) is a reference for upstream collaboration practices; this file defines PureTerm’s actual boundaries.

## Documentation language

English is the default reading language for every maintained Markdown document. Keep a complete Chinese translation in the paired `*_zh.md` file in the same directory. Link the Chinese file from the English file and update both files when behavior, commands, paths, or limits change. Code blocks, identifiers, links, and version numbers must remain equivalent. The standard MIT `LICENSE` text remains the canonical legal text in English.

## Current facts and historical material

- Current entry points are `apps/desktop/` and `apps/web/`; shared capabilities are in `packages/host/`, `packages/protocol/`, `packages/transport/`, and `packages/ui/`.
- Electron starts an independent Node Host child process for Desktop; standalone Web assembles Host in its own ordinary Node process.
- The root `package-lock.json` is the only lockfile. Run all install, build, and verification commands from the repository root.
- Current behavior is authoritative in the root `README.md`, `LAYOUT-PROPOSAL.md`, `VERSION.txt`, `docs/architecture.md`, `docs/DEVELOPMENT.md`, `docs/desktop-release.md`, the application READMEs, and `CHANGELOG.md`.
- Dated files under `docs/superpowers/` are historical implementation records and specifications. They may preserve durable criteria, correct advice, and explicitly rejected options, but do not treat them as current commands, paths, branches, or test results. Removed archive, review, and screenshot research material is not a current source.
- Screenshot and mouse/keyboard drivers are not product runtime code or verification entry points. Do not reintroduce the deleted `tools/gui/` directory or related research into build, test, or release flows.

## Repository layout

```text
apps/desktop/       Electron shell, runtime, Host child entry, carriers, and Desktop tests
apps/web/           standalone local Web Node entry, server, and tests
packages/host/      Cordis Host, SSH/SFTP, host storage, and credential interfaces
packages/protocol/  environment-neutral requests, events, capabilities, and binary protocol
packages/transport/ dispatcher, HTTP/WebSocket, carriers, and readiness validation
packages/ui/        Cordis Client, terminal, host list, SFTP, and browser adapters
VERSION.txt         source version baseline shared by all workspaces
scripts/            root workspace build, type, boundary, staging, and release checks
docs/               current architecture/release docs and dated historical records
.github/workflows/  three-platform Desktop build and GitHub Releases draft workflow
```

Organize directories by entry point and capability. Do not copy deepseek-harness’s scale by adding nested package groups, Agents, dynamic npm plugins, or a multi-tenant model. A new package must have an independent responsibility, consumers, and a verification boundary.

## Dependencies and boundaries

- `@pureterm/protocol` has no dependency on local packages, Electron, Node, or the UI.
- `@pureterm/host` does not depend on Electron, the UI, or application entry points; its public API is exported from the package entry.
- `@pureterm/ui` targets browsers only and cannot import Node, Electron, or Host; the page uses a static Cordis Client plugin composition.
- `@pureterm/transport` dispatches through the public Host API and cannot read `Host.internals`.
- Electron APIs may enter only Desktop `electron/app/`, `electron/carriers/`, preload, and diagnostic adapters; `electron/runtime/` and `electron/host/` have no Electron import.
- Cross-workspace references use public package exports; relative source imports are for modules inside one package.
- Keep source checks separate from artifact checks. Tests that require `dist/` must build first so stale artifacts cannot hide source errors.
- A protocol or public-type change in a shared package updates every consumer, test, document, and `CHANGELOG.md`; changing only the provider is incomplete.

## Runtime invariants

- The Desktop Host child process performs a versioned private RPC handshake. On startup failure, window close, renderer crash, update, or exit, the parent waits for Host cleanup and terminates only after the timeout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puremixai/pureterm](https://github.com/puremixai/pureterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
