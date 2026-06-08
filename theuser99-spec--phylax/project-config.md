---
trigger: always_on
description: This file is for AI coding agents (Codex, Claude Code, Cursor, Copilot, etc.) so they can understand the architecture before making changes.
---

# AGENTS.md - Phylax Codebase Guide

This file is for AI coding agents (Codex, Claude Code, Cursor, Copilot, etc.) so they can understand the architecture before making changes.

## What this project is

Phylax is a Windows security layer that constrains what AI agents can read, write, or delete at the OS level.

Primary stack: Rust.
Kernel minifilter driver (Phase 2): C++ in `driver/`.
Do not modify `driver/` unless explicitly requested.

## Workspace structure

```text
crates/
  agentguard-core/      <- Base types and shared errors (no external deps)
  agentguard-manifest/  <- phylax.toml parser + compiled GlobSets + auto-discovery
  agentguard-policy/    <- Decision engine (deny > ask > full > delete > write > read)
  agentguard-store/     <- SQLite access and schema ownership
  agentguard-probe/     <- Process polling + subject classification (Windows-focused)
  agentguard-enforce/   <- ACL/ACE enforcement and coordination
  agentguard-ipc/       <- Named-pipe protocol and client/server
  agentguard-notify/    <- User prompts/notifications for [ask]
  agentguard-audit/     <- Audit logging integration
  agentguard-daemon/    <- Main orchestrator/service logic
  agentguard-cli/       <- CLI entrypoint and commands
  agentguard-tui/       <- Ratatui dashboard
  agentguard-mascot/    <- Optional terminal mascot UI crate

crates not in workspace members:
  agentguard-spawn/     <- Standalone helper crate (present in repo, not listed in workspace members)

modules/
  agentguard-scanner/   <- Phase 3 placeholder
  agentguard-team/      <- Phase 4 placeholder

driver/
  C++ minifilter (Phase 2)

docs/adr/
  Architecture Decision Records
```

## Rules you must follow

### 1. Preserve dependency direction

```text
core <- manifest <- policy <- (enforce, audit, probe, notify, ipc) <- daemon
                                                              <- cli
                                                              <- tui
```

- `agentguard-core` must not depend on other workspace crates.
- `agentguard-manifest` and `agentguard-policy` should stay portable.
- All DB operations go through `agentguard-store`.

### 2. Do not change core types casually

`agentguard-core` is the contract between crates. Any change there can break the whole workspace.

### 3. Canonicalize paths before glob matching

```rust
let canonical = std::fs::canonicalize(&path)?;
let relative = canonical.strip_prefix(&workspace_root)?;
compiled_manifest.evaluate(relative, &op);
```

Never evaluate raw unresolved paths directly.

### 4. Store is the only DB boundary

No other crate should import `rusqlite` directly for business data access.

### 5. Test before claiming behavior

Use targeted crate tests first, then broader runs.

### 6. Phase 3/4 modules must remain decoupled

`modules/agentguard-scanner` and `modules/agentguard-team` should not depend on enforcement/probe/daemon internals.

## Permission model

Priority order:

1. `deny`
2. `ask`
3. `full`
4. `delete`
5. `write`
6. `read`

`deny` always wins.

Default when no rule matches:
- `conservative`: read Allow, write Ask, delete Deny
- `unrestricted`: Allow all

## IPC protocol snapshot

Current protocol includes 20 request types in `agentguard-ipc` (`RegisterProject`, `UnregisterProject`, `ValidateProject`, `CheckFileAccess`, `GetStatus`, `Shutdown`, `ReloadPolicy`, `AskResponse`, global rule CRUD, protection toggle, event subscription, stats/policy queries, and agent rule CRUD).

## Verified status (as tested on 2026-05-29)

Workspace members from root `Cargo.toml`:
- `agentguard-core`
- `agentguard-manifest`
- `agentguard-policy`
- `agentguard-store`
- `agentguard-probe`
- `agentguard-enforce`
- `agentguard-ipc`
- `agentguard-notify`
- `agentguard-audit`
- `agentguard-daemon`
- `agentguard-cli`
- `agentguard-tui`
- `agentguard-mascot`

Test listing counts (`cargo test -p <crate> -- --list`):

| Crate | Listed tests |
|---|---:|
| agentguard-core | 5 |
| agentguard-manifest | 48 |
| agentguard-policy | 8 |
| agentguard-store | 16 |
| agentguard-probe | 27 |
| agentguard-enforce | 11 |
| agentguard-ipc | 28 |
| agentguard-notify | 1 |
| agentguard-audit | 5 |
| agentguard-daemon | 22 |
| agentguard-cli | 38 |
| agentguard-tui | 0 |
| agentguard-mascot | 1 |

## Landing page (`landing/`)

The landing page is an Astro 4 static site in `landing/`. It deploys to GitHub Pages via `.github/workflows/deploy-landing.yml`.

**Stack:** Astro 4 + Tailwind CSS 3 + GSAP + Lenis smooth scroll.

**Key rules:**
- Do NOT commit `node_modules/`, `dist/`, or `.astro/` (see `landing/.gitignore`)
- All internal links must use `/Phylax/` prefix for GitHub Pages
- Build with `npm run build` from `landing/`

## Verified status (as tested on 2026-05-29)

Execution notes from this environment:
- `cargo test --workspace` is currently not fully green in this shell context.
- `agentguard-cli` e2e tests fail when daemon pipe `\\.\pipe\agentguard` is not running.
- `agentguard-enforce` has permission-sensitive tests that can fail with `SetNamedSecurityInfoW DACL: 5` depending on execution privileges.

## Useful commands

```bash
cargo build --workspace
cargo test --workspace
cargo test -p agentguard-manifest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheUser99-spec/Phylax](https://github.com/TheUser99-spec/Phylax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
