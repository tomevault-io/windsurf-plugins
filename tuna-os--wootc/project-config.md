---
trigger: always_on
description: - **`docs/status.md#buildtest-matrix`** — the one current status source
---

# wootc — Agent Guidance

## Start here

- **`docs/status.md#buildtest-matrix`** — the one current status source
  (the roadmap itself is `ROADMAP.md`).
  Green/red per image family × phase and per axis, from the E2E rig and the
  hosted matrix; a cell is only green once the hosted E2E passed it
  end-to-end (see `docs/RELEASING.md`). Open work is tracked on the org
  **TunaOS GA Roadmap** project (Tier 1 = silent-failure class, Tier 2 =
  user-facing correctness, Tier 3 = harness + process).
- **`docs/milestones.md`** — the verification ladder the matrix rolls up.
- **`docs/agent-lessons.md`** — traps that have each cost a 60–90 minute VM run.
  Read before touching the E2E harness, the deployer, or the runners.
- **`docs/docs-truth-pass.md`** — the dated record of each docs-vs-build pass
  (#233): what was checked, what was wrong, and what is still owed to an RC
  walk. Every ✘ it found is pinned by `tests/unit/docs-truth.bats`, so change
  a path, a default image, or a channel gate and that suite tells you which
  doc you just falsified.
- **`docs/phase2-debug-plan.md`** — *historical* (2026-07-18): its Phase-2
  hypotheses were resolved; the debugging story is in
  `docs/phase2-attach-postmortem.md`.

The single most useful heuristic in this codebase: **status derived from a proxy
rather than an observable is the dominant bug class here.** When adding a check,
ask what it would print if the thing it asserts never happened — then break the
code and confirm the test goes red.

## Project layers

The codebase has four distinct layers. Delegate to the matching agent when
working in a specific layer.

| Layer | Agent | Key files |
|-------|-------|-----------|
| Windows OEM | `wootc.wootc-windows-oem` | `autounattend.xml`, `setup-wootc.ps1`, `install.bat` |
| QGA control plane | `wootc.wootc-qga-control` | `qga.py`, QGA socket wiring in `compose.yml` |
| Deployer initramfs | `wootc.wootc-deployer` | `module-setup.sh`, `deploy.sh`, `deploy-hook.sh` |
| E2E test runner | `wootc.wootc-e2e-runner` | `run-e2e.sh`, Kanpur infrastructure |

All four agents have the `wootc-e2e` skill loaded, which covers shared
knowledge: PowerShell safety rules, QGA primitives, Kanpur quirks, and the
debug cycle.

## Project status

### QGA control plane — Live (commit 377a2ff)

The E2E control plane has been migrated from WinRM to QEMU Guest Agent.
The QGA client (`tests/e2e/qga.py`, 131-line stdlib Python) talks
JSON-lines over a virtio-serial Unix socket at `/run/shm/qga.sock`.

QGA provides:
- `guest-ping` for **generic liveness** (no credentials needed) — but it
  answers for whichever agent is up, Windows or Linux, so it cannot tell
  them apart on its own
- `guest-exec` for running PowerShell as SYSTEM (Windows) or `/bin/sh`
  (deployer / Phase-2 Linux)
- `guest-file-read` for reading OEM logs and the deployer journal
- Reboot detection via guest-ping down/up cycle

**Liveness ≠ identity.** Before every OS transition the runner asserts
identity positively: `$env:OS` must match `Windows_NT`
(`qga_windows_probe`) or `uname -s` must say Linux (`qga_linux_probe`) in
`tests/e2e/run-e2e.sh`. Relying on `guest-ping` alone has cost runs by
answering for the wrong guest.

See `docs/e2e-architecture.md` for the current control-plane topology;
`HANDOFF.md` is the historical (2026-07-15) design rationale.

### E2E Testing — autounattend.xml v3 Fixed (commit 0779d8d)
The critical fix: autounattend.xml was missing a `DiskConfiguration` block.
Without it, Windows Setup waits forever at "Where do you want to install
Windows?" — disk never grows past 1.2MB. v3 adds explicit UEFI GPT
partitioning (EFI 100MB + MSR 16MB + Primary), removes EnableLUA=false
(breaks Windows 11 boot), and merges WinRM setup into consolidated
FirstLogonCommands.

### BCD Chainload — Proven Working
`bcdedit /copy {bootmgr}` (not `/create /application firmware`) is the
correct approach. `/application firmware` is not a valid bcdedit type on
Windows 11. Fixed in `setup-wootc.ps1` and `app/installer_windows.go`.

### wubildr.efi — Built and Tested
Custom GRUB core image (1.3MB) with embedded bootstrap config, ntfs +
loopback modules. Built via `grub2-mkimage` inside the deployer container.
Stock Fedora grubx64.efi drops to rescue shell — wubildr.efi fixes this.

### Secure Boot chainload — shim + signed grub ✅ (commit 8a58274)

`wubildr.efi` is **unsigned**, so Secure Boot rejects it with `Access Denied`
on the serial console. The fix is a Microsoft-signed intermediate bootloader:

**UEFI → shimx64.efi → grubx64.efi → grub.cfg → deployer**

| Component | Signer | Source |
|-----------|--------|--------|
| `shimx64.efi` | Microsoft | Fedora `shim-x64` package |
| `grubx64.efi` | Fedora (in-shim MOK) | Fedora `grub2-efi-x64` package |
| `grub.cfg` | N/A (on ESP) | same logic as `wubildr.cfg` |

Fedora's signed `grubx64.efi` does NOT embed ntfs+loopback modules, but the
deployer and Phase 2 kernels + initramfs live on the FAT32 ESP — GRUB can
natively read FAT32 and never needs NTFS modules. The kernel command line
passes `loop=/wootc/disks/root.disk wootc.host_uuid=…` so the initramfs
(not GRUB) mounts the NTFS volume and attaches the loop device. No `.mod`
files are needed on the ESP.

### E2E boot chain progress

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuna-os/wootc](https://github.com/tuna-os/wootc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
