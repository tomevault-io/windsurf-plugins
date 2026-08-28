---
trigger: always_on
description: This file governs the entire repository.
---

# AGENTS.md

This file governs the entire repository.

## Product contract

Sandy is a macOS-native process sandbox for AI coding agents.

- Cargo workspace: `sandy-core`, `sandy-seatbelt`, and `sandy-cli`
- Installed executable: `sandy`
- Default mode: standalone sandboxing
- Optional integrations: preserve verified existing Kontext hooks or
  ownership-marked Numbat hooks; their explicit flags require them
- Explicit host setup: `sandy integrations setup kontext|numbat --agent NAME`
  may install and configure the selected optional provider
- Runtime model: one foreground supervisor per invocation, never a Sandy daemon

Sandy is a process sandbox, not a container or VM. Describe its guarantees
narrowly.

Version `0.1.x` is limited to macOS, one foreground `run` mode, Claude Code,
Codex, OpenCode, and generic profiles, explicit filesystem grants, network
allow/block with exact runtime Unix-socket and IPv4 local-host TCP exceptions,
dry-run output, and optional self-serve Kontext and host-installed Numbat hook
compatibility. It also includes an explicit foreground integration setup
command for Kontext and Numbat. Setup is not part of sandbox launch.

Agent presets are versioned, strictly typed profile documents embedded in the
CLI at compile time. Profiles resolve through deterministic inheritance in
`sandy-core` and may express only existing typed capabilities. Adding an agent
requires a profile document, an embedded registry entry, and tests; it must not
require renderer or bootstrap changes.

Do not add Linux, detached sessions, a PTY proxy, domain filtering, credential
brokering, dynamic grants, rollback, resource limits, raw Seatbelt input, or
organization-managed Kontext support or outside-sandbox synchronous hook
decision services without an explicit scope decision.

Do not modify the separate Kontext or Numbat repositories as part of Sandy
changes.

## Architecture

Use a virtual workspace with three packages representing validation,
native-code, and product boundaries.

```text
crates/core/               package sandy-core; validated security contract
crates/seatbelt/           package sandy-seatbelt; macOS compiler and FFI
crates/cli/                package sandy-cli; sandy binary and product UX
```

Do not add more crates until a distinct owner, dependency direction, and second
consumer or security boundary exists. Runtime-control resolvers and test support
remain modules inside `sandy-cli` in `v0.1.x`.

Keep dependencies flowing in one direction:

```text
CLI
  -> sandy-core
  -> sandy-seatbelt -> sandy-core

optional integrations
  -> typed capabilities
  -> never raw Seatbelt source
```

`sandy-core` performs deterministic validation but no ambient filesystem
discovery. `sandy-seatbelt` receives only validated policy and does not see
argv, environment, agent preset names, Clap, or service configuration. The CLI
does not render policy.

## Execution model

`sandy run` resolves the complete launch in the trusted parent, creates a
private session directory, and spawns the same executable in a hidden bootstrap
mode through `std::process::Command`.

The fresh bootstrap validates and removes a bounded, versioned manifest,
applies Seatbelt, and replaces itself with the target only after the sandbox
succeeds. Failures are reported on standard error without executing the target.

Do not use Rust `pre_exec` callbacks or run general Rust code in a
fork-after-threads child. The hidden bootstrap must not appear in normal CLI
help.

The parent remains outside the sandbox, supervises only the launched session,
cleans up session resources, and returns the target's exact exit status.

## Security invariants

These rules are release-blocking:

- The target never runs when resolution, validation, probing, rendering, or
  Seatbelt application fails.
- Unsupported and incompatible nested-sandbox environments fail closed.
- Sandy never falls back to unrestricted execution.
- Restrictions are inherited by every target descendant.
- The CLI and profiles accept typed capabilities, never raw Seatbelt rules.
- One centralized renderer validates and escapes every value used in policy.
- Paths are absolute, canonicalized, bounded, and compared as `Path`
  components rather than string prefixes.
- Canonicalization does not remove time-of-check/time-of-use risk; symlink and
  replacement behavior requires negative tests.
- Security configuration load failures are fatal. Never use a permissive
  default for missing protection data.
- Sensitive terminal deny rules override broader grants.
- Sandy-owned bootstrap resources must not survive target execution. Document
  that caller-supplied, non-`CLOEXEC` descriptors remain inherited capabilities.
- Give each session a mode-`0700` private `TMPDIR`; do not grant broad
  temporary-directory access.
- Strip `DYLD_*`, `SSH_AUTH_SOCK`, and security-routing overrides unless a
  reviewed capability explicitly requires them.
- Do not silently grant the home directory, Keychains, SSH material, Docker
  sockets, agent sockets, or unrelated local services.
- Filesystem access to a Unix socket never implies connect authority. Exact
  socket connections require a separate typed capability.
- A local-host TCP exception names one nonzero port and connect operation on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kontext-security/sandy](https://github.com/kontext-security/sandy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
