---
trigger: always_on
description: > ## ⚠ SUPERSEDED IN PART — read `RE_EXPERIMENT_PROCESS_CORRECTIONS.md` FIRST
---

# Clean-Room RE: Apple A18 Pro GPU Userspace (Apple9)

> ## ⚠ SUPERSEDED IN PART — read `RE_EXPERIMENT_PROCESS_CORRECTIONS.md` FIRST
>
> That document (repo root, user-authored 2026-08-30) is **normative for every new ISA/capability
> experiment and for any future evidence promotion**, and it says so itself: *"Where a single field
> label or a mechanical promotion rule conflicts with this document, this document wins."*
>
> **What it changes, in one paragraph.** Promotion now requires **five gates**: (A) an actual-byte
> ledger proving the value requested is the value really dispatched; (B) a pre-registered
> detection-power control per arm — a failed control means `carrier-undecidable`, never "inert";
> (C) an **independent semantic predictor**, because a difference from baseline is not an oracle and
> cross-run agreement proves repeatability rather than meaning; (D) a **generated recipe with no
> donor field**; (E) clean confirmation on a **measured-quiet** machine. One label may no longer
> carry four conclusions — score **six independent axes** (geometry, liveness, semantics, recipe,
> target, reproducibility). **`sem_checked == 0` can never produce `hardware-run`.**
>
> **`validate_labels.py` is NOT the promotion gate** (it validates schema, not evidence), and **no
> single `N of 166 emittable` headline may be derived from field labels at all** — the seven
> monotonic dashboards in `tools/agx-isa/dashboards.py` are the accounting, and
> `tools/agx-isa/promotion_check.py` is the gate.
>
> **§9/§10 matter as much as the gates:** never bulk-withdraw because a shared tool had a defect —
> re-read raw and scope the affected cases; and a later, stricter gate does **not** make an earlier
> observation false. Record both whether an experiment passed **its own frozen gate** and whether
> its evidence meets today's.

## Mission

Produce **clean-room hardware documentation** of the userspace-visible side of the
**Apple9-generation** AGX GPUs — primary documentation target **Apple A18 Pro (SoC T8140,
G17P)**, with the **local Apple M4 (G16G)** as the comparison, validation, and current
operational-execution target — sufficient for a *separate* implementation team to add support
to the Mesa `asahi` driver **after** the kernel driver (being built in parallel, out of scope
here) is in place.

We are the **reverse-engineering / documentation team**. We do **not** write the Mesa
driver. We write hardware specs; someone else implements them. This split is the core of
the clean-room defense.

**Operating contract.** `CODEX.md` is the binding process contract for every experiment
(the 10-step loop, evidence labels, minimum experiment record, provenance audit). The
authoritative task list / gap analysis is `APPLE9_RE_IMPLEMENTATION_GAPS.md`; the live status
board for the current goal is `docs/P0-P1-CLOSURE.md`. The acceptance bar for A18/M4 completeness is the
**unchanged Asahi UAPI** and its existing userspace/kernel division of responsibility — do
not classify something as kernel-managed merely because it was not visible in one capture;
check what the current UAPI requires userspace to supply.

**Current active goal: close all sixteen P0/P1 rows** (P0.1–P0.8, P1.1–P1.8) in
`docs/P0-P1-CLOSURE.md`, as defined by `APPLE9_RE_IMPLEMENTATION_GAPS.md` ( DRV-UAPI-01…04,
DRV-CMD-01, DRV-ISA-01, DRV-SHADER-01, DRV-ABI-01, DRV-PBE-01…DRV-RASTER-01; plus its P2, DOC,
and Part-II compiler-questionnaire tail). Execution strategy — **REVISED 2026-08-28**: **all new testing runs on the A18 Pro / G17P**
(`users-MacBook-Neo.local`, `192.168.170.254`). The earlier directive making the M4
the sole target is superseded: local GPU work destabilized the host it ran on. **Closure is now
measured against full G17P**, the actual documentation target. Committed M4/G16G evidence stays
valid on its own target and is not retracted — but new evidence lands on G17P, which converts a
large `INFERRED` debt into direct observation. Every result still records the target it actually
ran on; no cross-target promotion without a recorded validation or an explicit `INFERRED` label.

**Phase status (do not redo, do not contaminate):**
- **A18 Pro base documentation** (`docs/`, `tools/agx-isa`, `EXP-0001…0046`) — complete;
  the Apple9 baseline. `docs/ROADMAP.md` is its (historical) status board.
- **M4 validation** (`docs/ROADMAP-M4.md`, `docs/m4-deltas.md`, `EXP-M4-*`) — complete:
  every subsystem a driver emits is byte-identical A18↔M4; deltas are device
  identity/capacity only (`applegpu_g16g`, `AGXAcceleratorG16G`, 10 cores,
  `maxBufferLength` ~8.88 GiB — query, don't hard-code).
- **M5 (G17g, T8142)** — **goal complete** (`docs/ROADMAP-M5.md`, `EXP-M5-*`,
  `tools/agx-isa-m5`); a separate, later workstream. **M5 is deferred unless the user
  explicitly brings it into scope. Do not treat M5 results as evidence for A18/M4.**
- **Apple9 P0/P1 closure** (`APPLE9_RE_IMPLEMENTATION_GAPS.md` → `docs/P0-P1-CLOSURE.md`,
  `EXP-0047…` continuing) — **the active workstream.** New experiments take the next
  sequential `EXP-NNNN` number.

---

## THE PRIME DIRECTIVE: CLEAN ROOM ABOVE ALL ELSE

**It is better to have NO result than a TAINTED result.** A single tainted artifact can

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ADevWithAnIdea/agx-re](https://github.com/ADevWithAnIdea/agx-re) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
