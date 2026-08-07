---
trigger: always_on
description: Read this first. This root file contains universal safety and repository rules.
---

# AGENTS.md - mister-slint

Read this first. This root file contains universal safety and repository rules.
Subsystem-specific entrypoints and checks live in the nearest `AGENTS.md`.

## Critical Boot-Loop Safety

Highest priority: never leave the MiSTer in an unattended or persistent reboot
loop. A fast reset loop can make SSH unusable and may require pulling the SD
card to recover.

Never use persistent `launcher.env` to arm destructive reset faults.
`direct-reset-no-sync` must require a volatile `/tmp` session token. Cleanup
and exit traps for destructive runners must remove:

- `/media/fat/mister-magik/launcher.env`
- `/media/fat/mister-magik-dev/launcher.env`
- `/tmp/mister-magik/fs-fault-launcher.env`
- `/tmp/mister-magik/fs-fault-session`
- `/tmp/mister-magik/fs-fault.json`
- `/media/fat/mister-magik/rebuild-on-next-boot`
- `/media/fat/mister-magik-dev/rebuild-on-next-boot`

Host wait/recovery loops must use bounded local timeouts. Before any reset-fault
test, confirm a non-network recovery path and interruption-safe cleanup. After
direct-reset-no-sync experiments, verify no live arming file remains:

```bash
scripts/agent device arming-status
```

A single bounded recovery reboot is not a reboot loop. `scripts/agent diagnose`
may clear the listed arming files and issue one raw Linux reboot over SSH when
the installed platform is coherent and the launcher or its heartbeat is down.
That reboot request must never be automatically replayed.

If the MiSTer repeatedly reboots, stop normal deploy attempts. Remove stale
arming files first; if SSH is unstable, power down, mount the SD card on the
Mac, remove them directly, and inspect
`/media/fat/mister-magik/bootlogs/main-reboot.log`.

## Product And Canonical Names

MiSTer MagiK is a Rust/Slint frontend for MiSTer FPGA. The maintained
Main_MiSTer fork is normally at `../Main_MiSTer`; override with
`MISTER_MAIN_DIR`.

- Product/UI text: **MiSTer MagiK**
- Main binaries/processes: `MiSTer_MagiK`, `MiSTer_MagiKDev`
- Directory/script slug: `mister-magik`
- Slint binary/package: `mister-magik-fb`
- Rust crate/import: `mister_magik_fb`

Do not introduce the retired `magic` spelling or mixed-case path variants.

## Repository Routing

- `apps/mister/` — device frontend; read `apps/mister/AGENTS.md`
- `apps/mister/src/ui_runner/` — launcher runtime; read its local `AGENTS.md`
- `agent-cli/` — unified workflow and typed device tool
- `mister/tools/agent/` — device agent; read its local `AGENTS.md`
- `apps/desktop/` — macOS companion; read `apps/desktop/AGENTS.md`
- `scripts/` — validation/deploy/benchmark tooling; read `scripts/AGENTS.md`
- `private/magik-cloud/` — private submodule; read its local `AGENTS.md`
- `docs/` — current engineering policy
- `history/` — dated evidence, not current policy unless linked
- `reference/` — optional read-only research clones

Routine `rg` searches skip history, references, vendored dependencies, and
generated/build output through `.ignore`. Use `rg --no-ignore` explicitly when
those trees are part of the task.

## Universal Workflow Rules

- Preserve user changes. Never reset, checkout, clean, or overwrite unrelated
  work.
- Never amend commits on `main` after they have been pushed. Once history is
  published, add a new commit instead. Rewriting pushed branch history, including
  force-push or amend-and-force-push, requires an explicit user request and a
  clear statement of the remote state being replaced.
- Never use the Codex GitHub plugin for repository, issue, PR, or Actions work.
  Use `gh`.
- Agents use `scripts/agent deliver`, `benchmark`, or `diagnose` for device
  workflows. Diagnosis owns bounded read-only retries and one unattended
  one-shot recovery reboot. Attended operator operations use typed
  `scripts/agent device` commands; never raw SSH/SCP or generic remote-shell
  orchestration.
- Device workflows, Apple container, virtualization, and attended `mister`
  commands require first-attempt escalation using their direct repository
  command.
- Retry an explicitly read-only typed request once after a transient timeout,
  refusal, or route failure. Never blindly replay mutation: use the owning
  workflow's reconciliation or compensation path. Authentication and access
  failures require changed credentials or permissions before retrying. Report
  the device unavailable only after the bounded recovery path fails.
- Edit `MiSTer.ini` only through typed `mister` mutators or approved
  install/restore scripts.
- Apple Silicon ARM builds use Apple `container` by default. Do not switch to
  Docker/OrbStack.
- RBF synthesis runs only in the `Build MiSTer MagiK Platform` GitHub Actions
  workflow. Never attempt a local Quartus/RBF build or retain local RBF output.
- Enable `.githooks/pre-commit` with
  `git config core.hooksPath .githooks`.
- Treat `private/magik-cloud` as its own repository: commit and push it first,
  then update only the parent gitlink.
- Never stage private screenshots, caches, archives, `.env`, `.wrangler/`,
  credentials, or files under ignored `private/test-fixtures/`.
- Treat repos in `reference/` as read-only. However you can clone new repos into the folder.

## Top-Level Commands

```bash
scripts/agent plan
scripts/agent deliver
scripts/agent deliver local-main

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NigelBreslaw/MiSTer-MagiK](https://github.com/NigelBreslaw/MiSTer-MagiK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
