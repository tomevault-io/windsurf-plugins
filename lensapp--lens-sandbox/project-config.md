---
trigger: always_on
description: Lens Sandbox is a local desktop app for running AI agents, commands, OCI images, and other untrusted workloads inside a microVM. Tagline: "The sandbox you'll actually leave running." One-liner: "Run AI agents, commands, and OCI images locally. Control access into and out of the sandbox."
---

## Product Vision

Lens Sandbox is a local desktop app for running AI agents, commands, OCI images, and other untrusted workloads inside a microVM. Tagline: "The sandbox you'll actually leave running." One-liner: "Run AI agents, commands, and OCI images locally. Control access into and out of the sandbox."

- **Start here:** `docs/README.md` — the first-party user documentation index.
- **Product language:** the Product Vision above and `docs/` are the source of truth for terminology and framing; keep naming consistent with them. `docs/` is user-facing documentation only — no internal sales/marketing material lives in this repo.
- **Concepts:** approvals drive policy authoring; a per-directory `lns-policy.yaml` (auto-created with `defaultVerdict: ask`) holds network rules and custom credential-provider declarations, while per-machine credential value decisions are stored separately so secrets aren't committed; `Vz` on macOS / `KVM` on Linux is the only runtime; real secrets stay outside the workload via credential-shaped placeholders.
- **Sibling product:** Lens Agents is the centrally managed counterpart for IT teams. Same policy model.

Before proposing new features or architecture, consider whether they preserve the core principles: **a sandbox you don't turn off**, **ephemeral by default**, **no system dependencies** (the user runs one binary; no apt/brew preflight, no privileged installer), **policy you run into, not write**, **one directory = one project**, **real secrets stay outside the workload**. A small user-launched background service (the tray-resident `lns-service`, started by `lns service start` and stoppable via the tray Quit menu or `lns service stop`) is part of "a sandbox you don't turn off" — not a daemon in the apt/launchd sense.

## Project Overview

Monorepo. Eight Rust crates and one shell-script package today; built to grow.

| Package | Purpose |
|---------|---------|
| `crates/lns-cli` | The `lns` developer CLI — thin clap-driven IPC client that drives the daemon. The shipping artifact. |
| `crates/lns-service` | Tray-resident background service. Owns the microVM lifecycle, OCI ingest, content / layer caches, supervisor relay, and audit-chain writer; exposes a local Unix-socket IPC. |
| `crates/lns-ipc` | Shared `Request`/`Response` types and wire-format codec for the lns-cli ↔ lns-service contract. |
| `crates/lns-init` | Static-musl PID-1 for the guest microVM. Mounts composefs/overlay then `fexecve`'s `lns-session-broker`. |
| `crates/lns-session` | Wire-protocol types (postcard) for the host ↔ guest session channel. Shared by `lns-service` and `lns-session-broker`. |
| `crates/lns-session-broker` | Static-musl guest-side session host. `lns-init` execs into it; it owns PTY allocation, per-session workload forks, and vsock framing. |
| `crates/lns-supervisor` | Static-musl in-guest supervisor built on `lens-sandbox-core`. Embedded into `lns-service` and run inside the microVM; owns the agent process lifecycle, nftables network lockdown, privilege drop, and the vsock relay client. |
| `crates/bump-kernel` | Operator tooling for managing the kernel pin (`crates/lns-service/kernels.toml`). |
| `scripts/lns-install` | Installer shell script published to `get.lns.run`. |

## Conventions

- **Git**: Conventional commits (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `ci:`). No Co-Authored-By trailer. No "Generated with Claude Code" footer in PR descriptions.
- **Comments**: A comment is a code smell, not a feature — comments rot, code keeps moving, and stale comments are worse than none. Default is zero. Before adding any comment, first try to make the code carry the meaning: rename the binding, split the function, encode the invariant in a type, replace a literal with a named constant. Only when no refactor can convey the WHY does a comment go in, and then it is **one sentence, single line, no paragraph**. If it doesn't fit one sentence, the code is wrong, not the comment. The only categorical exceptions are `// SAFETY: <why this unsafe is sound>` on every `unsafe` block (clippy enforces) and `// no-op: <one-line reason>` above intentionally-empty Cucumber step defs. No "what is this" doc comments anywhere — `///` on internal items follows the same one-sentence-max bar; `pub` items on cross-crate API surfaces (`lns-ipc`, `lns-session`, exported traits) may carry one-sentence contract docs where the signature alone can't. No section-divider banners (`// =====`, `// ─────`). No `# Arguments` / `# Returns` boilerplate. No step-by-step narration inside function bodies — if a body needs a story, extract named helpers.
- **No prototype shortcuts**: avoid `unwrap()` outside tests, `unsafe` blocks without justification, suppressed lints, or papered-over errors. Fix things, don't paper over them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lensapp/lens-sandbox](https://github.com/lensapp/lens-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
