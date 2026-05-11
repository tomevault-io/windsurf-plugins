---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL: Credential Input Policy

**Never accept credentials (passwords, tokens, client secrets) via CLI flags or stdin.** This prevents exposure in shell history, `ps` output, and CI/CD logs.

- **Human credentials** (username, password): Interactive prompts only (`term.ReadPassword`). No flags, no env vars, no stdin.
- **Machine credentials** (token, client-id, client-secret): Environment variables (`JAMF_*`, `JAMFPROTECT_*`) for CI/CD. Interactive prompts for manual use. Config profiles with `keychain:` references for persistent storage. `--token-file` for file-based CI/CD.
- **Never add** `--password`, `--token`, `--client-secret`, `--token-stdin`, or `--client-secret-stdin` flags to any command.
- **Setup commands** (`pro setup`, `protect setup`, `config add-profile`) must always prompt interactively for credentials — no flag or env var bypass.

## CRITICAL: Generated Code Boundary

**Never edit files in `internal/commands/pro/generated/`** — they are overwritten by `make generate`.

**Never edit files in `internal/commands/platform/generated/`** — they are overwritten by `make generate`.

To change generated command behavior, edit the **generator templates**:
- **Modern API commands:** `generator/parser/generator.go` → `resourceTemplate` const
- **Classic API commands:** `generator/classic/generator.go` → `classicResourceTemplate` const
- **Modern registry:** `generator/parser/generator.go` → `registryTemplate` const
- **Classic registry:** `generator/classic/generator.go` → `classicRegistryTemplate` const

Templates are Go `const` strings embedded in the generator source — NOT separate `.tmpl` files.

After modifying a template: `make generate && make test`

## Platform Command Contract

Generated platform commands (`internal/commands/platform/generated/`) own **all** CRUD and action operations (list, get, create, update, delete, deploy, undeploy, report, etc.). Hand-written platform commands own **business logic only**: upsert (`apply`), portable export (`export`), profile conversion (`import-profile`), clone, dual-identifier lookup, and operations that orchestrate multiple API calls.

**Rule:** if a new Platform API endpoint maps cleanly to a single HTTP call, it should be a generated command, not hand-written. Wire generated subcommands under the hand-written parent `*cobra.Command`:

```go
for _, sub := range platformgen.NewBlueprintsCmd(cliCtx).Commands() {
    if sub.Name() == "create" { // skip if hand-written apply replaces it
        continue
    }
    cmd.AddCommand(sub)
}
```

CI enforces that `specs/platform/` and `internal/commands/platform/generated/` stay in sync: `make verify-platform-specs`. Run `make sync-platform-specs && make generate` after any spec change.

## Where to Make Changes

| I want to... | Edit this file |
|---|---|
| Change behavior of all modern API commands | `generator/parser/generator.go` (`resourceTemplate`) |
| Change behavior of all classic API commands | `generator/classic/generator.go` (`classicResourceTemplate`) |
| Change how OpenAPI specs are parsed | `generator/parser/parser.go` |
| Change singleton detection logic | `generator/parser/parser.go` → `detectSingleton()` |
| Change multi-family spec splitting | `generator/parser/parser.go` → `splitByPathFamilies()` |
| Add/change alternate lookup fields (--serial, --udid) | `generator/parser/parser.go` → `resourceLookupFields` map |
| Fix a resource name auto-pluralization issue | `generator/parser/parser.go` → `resourceNameOverrides` map |
| Fix wrong RSQL filter field for --name lookup | `generator/parser/parser.go` → `resourceNameFieldOverrides` map |
| Fix wrong ID field extracted from list response | `generator/parser/parser.go` → `resourceIDFieldOverrides` map |
| Change how classic YAML manifest is parsed | `generator/classic/parser.go` |
| Add a new resource to the classic API | `specs/classic/resources.yaml` |
| Add/modify DDM component scaffolds | `internal/blueprintcomponents/scaffolds.go` — SDK-typed components via `example*()` funcs; raw JSON fallback in `rawScaffolds` for components not yet in SDK |
| Add a new legacy-to-DDM payload converter | `internal/profileconvert/ddm_<name>.go` (new converter + register in `ddm_converter.go` init) |
| Add/remove a resource in the `backup`/`diff` commands | `internal/commands/pro_resources.go` (curated allowlist; endpoints come from generated `backup_registry.go`) |
| Add a new Jamf Pro handwritten command | `internal/commands/pro_*.go` (new file + wire in `pro.go`) |
| Add a new Platform API endpoint (CRUD, actions, reports) | Drop/update spec in `specs/.platform-source/`, run `make sync-platform-specs && make generate`. Don't hand-write — generator owns CRUD/actions. |
| Add a new Platform business operation (apply, import-profile, clone, etc.) | Hand-write in the relevant `pro_<resource>.go`; CRUD primitives must come from `internal/commands/platform/generated/` |
| Add or change a generated Platform API resource | drop a spec into `specs/.platform-source/*.json`, then `make sync-platform-specs && make generate` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jamf-Concepts/jamf-cli](https://github.com/Jamf-Concepts/jamf-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
