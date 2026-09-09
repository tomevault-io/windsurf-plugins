---
trigger: always_on
description: Instructions for GitHub Copilot and other AI coding agents working with the .NET SDK repository.
---

# Agent Instructions

Instructions for GitHub Copilot and other AI coding agents working with the .NET SDK repository.

## Overview

This is `dotnet/sdk`, the repository for the core of the .NET SDK. It produces the
`dotnet` CLI driver and the MSBuild tasks, targets, templates, and resolvers that are
shared between the .NET CLI and Visual Studio. The build output is a complete, runnable
`dotnet` installation laid down at `artifacts/bin/redist/<configuration>/dotnet`
(`Debug` by default).

For a high-level project description, build status, and contribution flow, see the
[README](README.md). For the canonical build/test/debug walkthrough, see the
[Developer Guide](documentation/project-docs/developer-guide.md).

### Agent orientation and memory

1. Read the [memory index](.github/memory/INDEX.md) first and load other memory files on demand.
2. For non-trivial work, also read [ARCHITECTURE.md](.github/memory/ARCHITECTURE.md) and
   [CONVENTIONS.md](.github/memory/CONVENTIONS.md).
3. Treat memory as orientation; cross-check important claims against linked primary sources.
4. Correct stale memory in the same change and keep the index synchronized.

### Grounding architecture and product claims

Treat this overview as an index, not as independent evidence. In plans, reviews,
root-cause analyses, and AI-facing documentation:

- Verify important architecture, product behavior, and ownership claims against primary
  sources and link the nearest evidence: code or project files for current behavior, and an
  ADR, design document, issue, or PR for decisions and history.
- Prefer repository-relative links for in-repo evidence and link the narrowest durable
  source. Do not cite this overview to support itself.
- When a cited in-repo source is relevant to the task, inspect it before relying on the
  claim; do not assume the link target's contents are already in context.
- Identify inference explicitly and cite its inputs. If sources disagree or evidence is
  incomplete, state the uncertainty instead of turning synthesis into fact; update stale
  context in the same change.

### What the SDK does

See [ARCHITECTURE.md](.github/memory/ARCHITECTURE.md) for product components and data flow,
[FILE_MAP.md](.github/memory/FILE_MAP.md) for repository locations, and
[API_MAP.md](.github/memory/API_MAP.md) for user-facing and extension surfaces.

### Repository boundaries and the VMR

An SDK command or build can expose behavior implemented by another .NET repository. Find
the component that defines the behavior before making a change; do not add an SDK
workaround merely because the symptom appears through `dotnet`, and do not infer ownership
from a diagnostic ID. See the canonical
[ownership map](.github/memory/ARCHITECTURE.md#ownership-boundaries).

### Build and test

- Build the redist SDK with `build.cmd` on Windows or `./build.sh` on Linux/macOS.
- Add `-test` / `--test` for the full suite and `-pack` / `--pack` for packages/installers;
  avoid these large operations in the routine inner loop.
- Use [`run-tests` skill](.github/skills/run-tests/SKILL.md) for focused validation and
  [`incremental-test` skill](.github/skills/incremental-test/SKILL.md) for supported
  `dotnet.Tests` changes.
- Product tests exercise `artifacts/bin/redist/<configuration>/dotnet`; ensure it contains
  the production change before trusting results.

| Switch | Effect |
| --- | --- |
| `-c` / `-configuration <Debug\|Release>` | Build configuration (default `Debug`). |
| `-test` (`-t`) | Run tests after building. |
| `-pack` | Build installers/packages (otherwise skipped for speed). |

Arguments not directly supported by the script are passed through to MSBuild (e.g.
`/t:UpdateXlf`, `/bl` for a binlog, `/p:Property=Value`).

Canonical scenarios:

- Build the full redist SDK: `build.cmd` (Windows) or `./build.sh` (Linux/macOS).
  - The script first restores a repo-local .NET SDK to `.dotnet/dotnet`, then builds the SDK.
    Invoke that bootstrap SDK directly as `./.dotnet/dotnet <args>` when you need a `dotnet`
    that resolves against this repo.
  - The built SDK is output to `artifacts/bin/redist/<configuration>/dotnet` (`Debug` by default).
  - The first build is slow; subsequent builds are incremental.
- Run tests through the `run-tests` skill. It selects the appropriate focused, scoped, or
  full-suite workflow and retains actionable diagnostics for focused runs (see
  [Testing](#testing)).
- Release build: `build.cmd -c Release`.
- Validate changes locally using the SDK you built at
  `artifacts/bin/redist/<configuration>/dotnet` (`Debug` by default).

See [TESTING_STRATEGY.md](.github/memory/TESTING_STRATEGY.md) and the
[Developer Guide](documentation/project-docs/developer-guide.md).

## Guardrails

These are hard boundaries for agents working in this repo. Treat them as "must not" rules.

### Flag user-visible behavior

Call out intentional user-visible behavior or contract changes in the final handoff.

### Do not hand-edit generated files

Some files are produced by tooling and are overwritten the next time the build or a
generation step runs. Editing them by hand causes drift and merge conflicts. Never
manually edit:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/sdk](https://github.com/dotnet/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
