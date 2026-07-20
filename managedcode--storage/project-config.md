---
trigger: always_on
description: Project: ManagedCode.Storage
---

Project: ManagedCode.Storage
Stack: .NET 10 / C# / xUnit / VSTest / Coverlet / GitHub Pages docs

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Purpose

This file defines how AI agents work in this solution.

- Root `AGENTS.md` holds the global workflow, shared commands, cross-cutting rules, maintainability limits, and global skill catalog.
- Every `.csproj` root in this multi-project solution keeps a local `AGENTS.md` with project-specific entry points, boundaries, commands, risks, and applicable skills.
- Local `AGENTS.md` files may tighten root rules, but they must not weaken them silently.

## Solution Topology

- Solution root: `ManagedCode.Storage.slnx`
- Projects or modules with local `AGENTS.md` files:
  - `ManagedCode.Storage.Core/`
  - `ManagedCode.Storage.TestFakes/`
  - `ManagedCode.Storage.VirtualFileSystem/`
  - `Integraions/ManagedCode.Storage.Client/`
  - `Integraions/ManagedCode.Storage.Client.SignalR/`
  - `Integraions/ManagedCode.Storage.Orleans/`
  - `Integraions/ManagedCode.Storage.Server/`
  - `Storages/ManagedCode.Storage.Aws/`
  - `Storages/ManagedCode.Storage.Browser/`
  - `Storages/ManagedCode.Storage.Azure/`
  - `Storages/ManagedCode.Storage.Azure.DataLake/`
  - `Storages/ManagedCode.Storage.CloudKit/`
  - `Storages/ManagedCode.Storage.Dropbox/`
  - `Storages/ManagedCode.Storage.FileSystem/`
  - `Storages/ManagedCode.Storage.Google/`
  - `Storages/ManagedCode.Storage.GoogleDrive/`
  - `Storages/ManagedCode.Storage.OneDrive/`
  - `Storages/ManagedCode.Storage.Sftp/`
  - `Tests/ManagedCode.Storage.Tests/`
  - `Tests/ManagedCode.Storage.BrowserServerHost/`
  - `Tests/ManagedCode.Storage.BrowserWasmHost/`

## Rule Precedence

1. Read the solution-root `AGENTS.md` first.
2. Read the nearest local `AGENTS.md` for the area you will edit.
3. Apply the stricter rule when both files speak to the same topic.
4. Local `AGENTS.md` files may refine or tighten root rules, but they must not silently weaken them.
5. If a local rule needs an exception, document it explicitly in the nearest local `AGENTS.md`, ADR, or feature doc.

## Conversations (Self-Learning)

Learn the user's stable habits, preferences, and corrections. Record durable rules here instead of relying on chat history.

Before doing any non-trivial task, evaluate the latest user message.
If it contains a durable rule, correction, preference, or workflow change, update `AGENTS.md` first.
If it is only task-local scope, do not turn it into a lasting rule.

Update this file when the user gives:

- a repeated correction
- a permanent requirement
- a lasting preference
- a workflow change
- a high-signal frustration that indicates a rule was missed

Extract rules aggressively when the user says things equivalent to:

- "never", "don't", "stop", "avoid"
- "always", "must", "make sure", "should"
- "remember", "keep in mind", "note that"
- "from now on", "going forward"
- "the workflow is", "we do it like this"

Preferences belong in `## Preferences`:

- positive preferences go under `Likes`
- negative preferences go under `Dislikes`
- comparisons should become explicit rules or preferences

Corrections should update an existing rule when possible instead of creating duplicates.

Treat these as strong signals and record them immediately:

- anger, swearing, sarcasm, or explicit frustration
- ALL CAPS, repeated punctuation, or "don't do this again"
- the same mistake happening twice
- the user manually undoing or rejecting a recurring pattern

Do not record:

- one-off instructions for the current task
- temporary exceptions
- requirements that are already captured elsewhere without change

Rule format:

- one instruction per bullet
- place it in the right section
- capture the why, not only the literal wording
- remove obsolete rules when a better one replaces them

## Global Skills

List only the skills this solution actually uses.

- `mcaf-solution-governance` — use when bootstrapping or refining root and local `AGENTS.md`, maintainability limits, rule precedence, or solution topology.
- `mcaf-architecture-overview` — use when creating or updating `docs/Architecture.md` after module, boundary, or contract changes.
- `mcaf-documentation` — use for durable docs, docs-site synchronization, Mermaid-heavy docs updates, and repo documentation structure changes.
- `mcaf-adr-writing` — use when documenting cross-cutting architectural or standards decisions in `docs/ADR/`.
- `mcaf-feature-spec` — use when documenting non-trivial feature behavior in `docs/Features/`.
- `mcaf-dotnet` — entry skill for .NET work and routing to specialized `.NET` skills.
- `mcaf-dotnet-analyzer-config` — use when the repo-root `.editorconfig` or analyzer severity ownership changes.
- `mcaf-dotnet-code-analysis` — use when SDK analyzer policy in `Directory.Build.props` or project files changes.
- `mcaf-dotnet-features` — use when modern C# or .NET 10 feature choices matter.
- `mcaf-testing` — use for scenario coverage planning and verification strategy.
- `mcaf-dotnet-netarchtest` — use when architecture dependency rules in `Tests/ManagedCode.Storage.Tests/Architecture/` change.
- `mcaf-dotnet-xunit` — use for xUnit tests in `Tests/ManagedCode.Storage.Tests/`.
- `mcaf-dotnet-quality-ci` — use for the repo quality pass and CI-aligned verification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [managedcode/Storage](https://github.com/managedcode/Storage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
