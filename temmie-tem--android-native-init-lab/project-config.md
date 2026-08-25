---
trigger: always_on
description: Contract-Revision: **2** (supersedes revision 1; 2026-08-03)
---

# AGENTS.md - repository operating contract

Contract-Revision: **2** (supersedes revision 1; 2026-08-03)

The retired Interim Fast-Loop trial contract is preserved byte-for-byte at `docs/archive/policy/AGENTS_INTERIM_FAST_LOOP_RETIRED_2026-08-03.md`; it is historical evidence only and grants no current authority.

---

This file contains the repository-wide invariants and the binding target registry.
Select exactly one target contract before target-specific work. A registry goal describes current state and objectives but never grants device authority.
Historical or draft policies under `docs/archive/` or elsewhere are evidence only, even if their text says `ACTIVE`.

The default work cycle is:

`STATE -> SELECT -> DESIGN -> IMPLEMENT -> STATIC VALIDATE -> DEVICE -> REPORT -> COMMIT`

Do not add a device step when host-only work can answer the question.

## Authority and Precedence

The effective contract is, in descending order:

1. the common invariants in this file;
2. the selected binding target contract in the registry below;
3. the shared risk-tier and execution-process documents named by that target;
4. the immutable live binding required by the active policy or current runner.

The more restrictive applicable rule wins. A target contract may specialize
only behavior explicitly delegated by this file and may never relax the
permanent safety boundaries. A manifest, approval string, goal, report,
archived clause, helper string, or sub-goal cannot override a higher layer.

No document grants standing device authority unless this common contract or the
selected target contract expressly activates it and all required live inputs
are current. An unactivated policy edit remains H0 only.

## Binding Target Registry

| Target | Current state | Binding target contract | Binding live process |
|---|---|---|---|
| Samsung Galaxy S22+ FYG8 (`SM-S906N` / `g0q` / `S906NKSS7FYG8`) | `GOAL.md` | `docs/operations/targets/S22PLUS_FYG8_TARGET_CONTRACT.md` | `docs/operations/DEVICE_ACTION_PROCESS_V2.md` |
| Samsung Galaxy A90 5G | `GOAL_A90.md` | `docs/operations/targets/A90_TARGET_CONTRACT.md` | `docs/operations/targets/A90_TARGET_CONTRACT.md` sections `A90 D1 Resident Session`, `A90 F1 Resident Install`, and `Attended F1 Pre-Handoff` |
| Samsung Galaxy S20+ 5G (`SM-G986N` / `y2q` / `G986NKSS8IYC2`) | `GOAL_S20PLUS.md` | `docs/operations/targets/S20PLUS_G986N_TARGET_CONTRACT.md` | Active exact-target routine D0/D1 including payload-free Download return; attended boot-only bootstrap and resident Magisk F1 active; reviewed attended native-canary R1 active |

Targets, profiles, rollback identities, transports, approvals, and health evidence never transfer between registry rows. Without an exact matching contract, remain H0.

For A90 work, read this file, then `docs/operations/targets/A90_TARGET_CONTRACT.md`, then `GOAL_A90.md`. The goal cannot grant or extend live authority.

## Permanent Device Safety Boundaries

1. Work only on an explicitly identified operator-owned device. Device effects
   require attendance except the exact A90 resident D1 lane and an exact S20+
   bounded autonomous-research lane separately activated by their binding
   target contracts. F1 is never unattended, and authority never transfers
   between targets.
2. The only partition payload permitted by the ordinary process is **boot**.
   Never send a partition image, raw block write, or flashing operation to
   recovery, vendor_boot, DTBO, vbmeta, vbmeta_system, BL, CP, CSC, super,
   userdata, persist, EFS, sec_efs, RPMB, keymaster, modem, bootloader, or any
   other partition. An exact reviewed D1 action performed through normal
   Android Package Manager or shared-user-storage APIs is an OS-mediated data
   write, not a partition payload; it is permitted only within the closed
   package/file staging rules below and never authorizes block or filesystem
   access to a partition mount outside that normal API.
3. Never use raw host `dd`, fastboot, partition-table actions, qdl/Sahara/
   Firehose, RAM dump, EUD/UART writes, fuse/QFPROM actions, format operations,
   or an unreviewed panic/RDX path.
   One narrow A90 boot-control exception may be activated by the A90 target
   contract: after an exact reviewed `boot` write and readback, the fixed TWRP
   System-reboot hook may clear exactly the first 256 bytes of `misc` BCB and
   nothing else before reboot. The hook path, complete bytes, size, SHA-256,
   TWRP version, recovery identity, ordering, and one-shot behavior must be
   fixed by reviewed code. It accepts no caller path, offset, count, command,
   or payload; drift or a second invocation stops. This exception grants no
   other `misc` access and never transfers to another target or process.
4. Never flash unless the exact rollback artifact is present, readable,
   hash-verified, and usable through a demonstrated recovery path.
5. Never flash a new experiment over an unhealthy or unverified device.
   Recover first, verify health, and stop that experiment.
6. A target ambiguity, unexpected archive member, forbidden partition signal,
   changed artifact, missing rollback, journal inconsistency, or lost physical
   recovery path is an immediate stop.
7. After an unexplained failure once a device or transfer session starts, stop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Temmie-Tem/android-native-init-lab](https://github.com/Temmie-Tem/android-native-init-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
