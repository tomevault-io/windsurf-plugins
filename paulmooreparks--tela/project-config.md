---
trigger: always_on
description: At the start of each conversation, read these files to understand the project
---

# CLAUDE.md - Tela Project Context

## Onboarding

At the start of each conversation, read these files to understand the project
context, current status, and design direction:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Build commands, coding conventions, API style, review items |
| `DESIGN.md` | Architecture, protocol spec, component design, roadmap |
| `DESIGN-remote-admin.md` | Agent/hub management protocol, implementation status |
| `DESIGN-file-sharing.md` | File sharing protocol and implementation |
| `DESIGN-portal.md` | Portal protocol specification (the wire contract every Tela portal must implement) |
| `STATUS.md` | Traceability matrix mapping design sections to implementation |
| `book/src/guide/telavisor.md` | TelaVisor desktop client (canonical chapter, comprehensive). The repo-root `TelaVisor.md` is a short overview pointing here. |
| `TELA-DESIGN-LANGUAGE.md` | Visual design language shared across all Tela products |
| `ACCESS-MODEL.md` | Token-based RBAC, permissions, ACL model |
| `CONFIGURATION.md` | Config file formats for all three binaries |
| `REFERENCE.md` | CLI reference for tela, telad, and telahubd |
| `RELEASE-PROCESS.md` | Channel model (dev/beta/stable), promotion, manifests |
| `ROADMAP-1.0.md` | 1.0 readiness checklist (blockers, important, polish) |
| `TODO.md` | Current task list and priorities |

Do not read all files upfront in every conversation. Read `CLAUDE.md` always.
Read the others when the conversation topic requires them (e.g., read
`DESIGN-remote-admin.md` when working on agent management, read
`book/src/guide/telavisor.md` when working on the GUI). Use your judgment
to decide which files are relevant.

## Project Overview

Tela is a FOSS encrypted remote-access fabric using WireGuard tunnels.
Three binaries, one Go module (`github.com/paulmooreparks/tela`):

| Binary | Path | Role |
|--------|------|------|
| `tela` | `cmd/tela/` | Client CLI (connects to machines via hub, mounts file shares) |
| `telad` | `cmd/telad/` | Agent daemon (registers machines with hub, exposes local services) |
| `telahubd` | `cmd/telahubd/` | Hub server (HTTP + WebSocket + UDP relay on a single port) |

Supporting packages:
- `internal/channel/` -- Release channel manifest fetcher, validator, and SHA-256 stream verifier. Used by every binary's self-update path.
- `internal/credstore/` -- User-level credential store (`~/.tela/credentials.yaml` on Unix, `%APPDATA%\tela\credentials.yaml` on Windows). Stores hub tokens and the client's own channel preference.
- `internal/service/` -- Cross-platform OS service management (Windows SCM, systemd, launchd)
- `internal/telelog/` -- Logging primitives shared across binaries
- `internal/wsbind/` -- WebSocket transport binding helpers
- `console/` -- Embedded static files for the hub's web console
- `cmd/telagui/` -- TelaVisor desktop GUI (Wails v2: Go backend + vanilla JS frontend)

GitHub Actions workflows:
- `.github/workflows/ci.yml` -- Build, vet, test, gofmt, mod tidy, cross-compile sanity check on every push and PR
- `.github/workflows/release.yml` -- Build matrix that produces dev/beta/stable releases. Compute-version job at the top reserves the tag atomically, downstream jobs all build against it. Publishes to a rolling `channels` GitHub Release.
- `.github/workflows/promote.yml` -- Manual `workflow_dispatch` that promotes a dev tag to beta or a beta tag to stable, then invokes release.yml via `workflow_call` to build the new tag.

Related project: **Awan Saya** (`c:\Users\paul\source\repos\awansatu`) -- portal/registry that discovers and lists hubs. Changes to tela's auth, portal sync, or API contracts may affect Awan Saya.

## Wirint Style

- Do not use emdash
- Do not use a style of writing that would even require emdash or semicolons.
- Do not use curly quotes, single or double. Use ' or " as appropriate instead.
- Do not use a salesy, marketing style of writing, unless instructed to do so. Simply be factual instead.
- Write in the style of a technical writer producing exact documentation, unless instructed otherwise.
- Be very clear and thorough in technical writing. Do not leave out steps.
- Spell out the first abbeviation usage in a document unless you can be reasonably sure the abbreviation is known in context (e.g., TCP and UDP are known, FOSS is not necessarily known)
- Capitalize all headings and subheadings using standard English Title Case. Do not use sentence case for headings. For example, write "What Tela Is" not "What Tela is", and "The Three Binaries" not "The three binaries". Articles ("a", "an", "the"), short prepositions ("of", "to", "in", "on", "at", "by", "for", "with"), and coordinating conjunctions ("and", "but", "or", "nor", "so") stay lowercase unless they are the first or last word. This applies to every heading in every markdown file, HTML page, design document, and inline comment section banner. The rule also applies to pull request titles and commit subject lines.

## Editorial Workflow (markdown sweeps)

The user acts as editor on all markdown files in this repo (the mdBook under
`book/`, plus top-level docs like `README.md`, `REFERENCE.md`, `DESIGN.md`,
etc.) by leaving inline HTML comments tagged with an `EDIT:` marker. These

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulmooreparks/tela](https://github.com/paulmooreparks/tela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
