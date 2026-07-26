---
trigger: always_on
description: This file applies to the root repository and its checked-out submodules unless a
---

# Dumplings Agent Guide

This file applies to the root repository and its checked-out submodules unless a
more specific `AGENTS.md` is added below a directory. It is an operating contract
for coding agents, not a replacement for the component READMEs or the workflows
under `.agents/skills`.

## Non-Negotiable Rules

- Use Windows PowerShell Core 7.4 or later. Run repository commands from the
  Dumplings root unless a command explicitly targets a submodule.
- Never execute an unknown or downloaded installer on the host. Analyze it
  statically. Use a checkpointed Windows Sandbox or Hyper-V VM when dynamic
  evidence is unavoidable.
- Never invent package metadata, legal names, registry values, silent switches,
  or installer behavior. Return unresolved evidence as a warning and validate it
  in the VM when necessary.
- Preserve unrelated user changes. Inspect the root and every affected submodule
  before editing; do not reset, revert, or reformat unrelated files.
- Keep secrets out of Git. `.env`, `Secret.yaml`, `Sandbox`, and `Outputs` are
  local state. Do not cite them as durable test fixtures or source evidence.

## Repository And Git Boundaries

Dumplings is a superproject with independently versioned repositories:

```text
Dumplings                         root project (MIT)
+-- Core                          git submodule (Apache-2.0)
+-- Modules/PackageModule         git submodule (Apache-2.0 with file exceptions)
`-- Modules/InstallerParsers      git submodule (file-specific GPL/MIT)
```

`git status` at the root reports a modified submodule only as a pointer change.
When touching a submodule, also run `git -C <submodule> status --short` and review
its own diff. Commit submodule changes in that repository before updating the
root pointer when the user requests commits.

Task state changes under `Tasks` are automation output and may be intentional.
Do not edit or discard them unless the task explicitly requires it. Avoid broad
formatting passes over thousands of task scripts.

## Start With The Existing Architecture

Read the nearest component README before changing its code:

- `README.md`: project operation and task basics.
- `Core/README.md`: runner, dependencies, hooks, workers, and synchronization.
- `Modules/PackageModule/README.md`: task model, manifest model, services, and
  in-process parser APIs.
- `Modules/InstallerParsers/README.md`: GPL CLI boundary and parser contracts.

Use the detailed skills for domain work:

- `.agents/skills/analyze-winget-installer/SKILL.md` for installer analysis,
  parser routing, ARP evidence, and VM validation.
- `.agents/skills/author-winget-manifest/SKILL.md` for source discovery,
  manifest and locale authoring, formatting, validation, and submission.

Do not duplicate their installer-family or manifest-field rules in general
documentation. Update the focused workflow when a newly learned rule affects
future authoring.

## PowerShell And Source Style

- Follow `.editorconfig`: UTF-8 without BOM, CRLF, final newline, no trailing
  whitespace, and two-space indentation for PowerShell, YAML, and JSON.
- Prefer `[ordered]` dictionaries where deterministic serialization matters.
- Add comment-based help to public functions. Document parameters, stream
  ownership, offset bases, units, and output contracts when they are not obvious.
- Comment meaningful parsing and failure paths, not individual assignments.
- Avoid introducing local copies of helpers already provided by `General.psm1`,
  shared binary/archive/PE infrastructure, or Core synchronization.
- Do not set module-wide `$ErrorActionPreference` merely to force helper errors.
  Use advanced functions, terminating errors where required, and common
  `-ErrorAction` behavior.
- Guard `Add-Type` by checking whether its type already exists. Modules are loaded
  repeatedly in tests and worker runspaces.
- Prefer typed lists, streaming output, and bounded streams for large data. Avoid
  accidental PowerShell `Object[]` materialization and unbounded `ReadAllBytes`
  on installer files.
- Keep functions deterministic and independently callable where practical. Core
  globals may be used only where the documented standalone fallback exists.

## Core, Hooks, And Concurrency

Core owns runner infrastructure. PackageModule behavior should normally be wired
through scripts under `Modules/PackageModule/Hooks`, not hard-coded into
`Core/Index.ps1`.

Available lifecycle phases include `RunnerStarting`, `WorkerStarting`,
`BeforeTask`, `AfterTask`, `WorkerStopping`, `BeforeForcedWorkerStop`, and
`RunnerStopping`. Cleanup hooks must be idempotent and release resources even
after task failures or worker timeouts.

Use `Use-Mutex`, `Use-Semaphore`, or `Use-Monitor` from
`Core/Libraries/Synchronization.psm1`. Do not construct raw synchronization
primitives in task scripts. Native `winget` calls must remain serialized because
concurrent invocations can crash the client.

`$Global:DumplingsStorage` is synchronized process-wide storage shared across
thread-job runspaces. `$Global:DumplingsSessionStorage` is runspace-local. If one
task supplies shared data to another, declare it with `DependsOn` in
`Config.yaml`; shared-storage access is not an implicit dependency.

## Package Tasks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpecterShell/Dumplings](https://github.com/SpecterShell/Dumplings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
