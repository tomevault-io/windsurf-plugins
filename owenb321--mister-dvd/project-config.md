---
trigger: always_on
description: This project is a **DVD player core for the MiSTer FPGA platform (DE10-Nano / Cyclone V)**,
---

# MiSTer DVD Player Core — Claude Code Context

## Project Overview

This project is a **DVD player core for the MiSTer FPGA platform (DE10-Nano / Cyclone V)**,
built as a fork of [`mrchrisster/MiSTer_MPEG2`](https://github.com/mrchrisster/MiSTer_MPEG2).

The goal is to extend the existing working MPEG-2 video decoder into a full DVD player,
adding Program Stream demuxing, UDF/IFO navigation, AC-3 and DTS audio passthrough/decode,
and CSS decryption — all while keeping the proven FPGA video pipeline intact.

See `docs/` for detailed reference on architecture, audio, and implementation roadmap.

---

## Documentation Discipline (read before starting work)

Every non-trivial design decision **must** be written down — either in this `CLAUDE.md`
or in the `docs/` folder. Code without recorded rationale is treated as incomplete.

- **Where to put it:**
  - `CLAUDE.md` — durable, project-wide rules, conventions, and high-level decisions an
    agent needs *before* touching code (architecture choices, toolchain pins, workflow).
  - `docs/architecture.md`, `docs/audio.md`, `docs/roadmap.md`, `docs/references.md` —
    detailed, subject-specific design notes, data flows, FSM descriptions, and rationale.
  - Per-module status (e.g. the `ps_demux.sv` "Status & design decisions" block below) —
    a short summary lives in `CLAUDE.md`, full detail goes in `docs/architecture.md`.

- **When to write it:** at the same time as the code, not "later." Commit docs together
  with the change that motivated them.

- **What to record:** the *why* behind each decision, known limitations / TODOs, design
  alternatives that were rejected (and why), and anything that surprised you or wasn't
  obvious from reading the code.

### Leave a trail to resume work in a new session

Sessions are stateless — after a feature branch merges, the next session starts cold and
has only the committed markdown to go on. Before finishing any feature, ensure the docs
leave enough hints to pick up cleanly:

- Update the relevant **status block** (what's implemented, what's wired in, what isn't)
  and the ✅/❌ checklists in "What Already Works" / "Known Gaps".
- Record the **next concrete step** so the following session knows where to start
  (e.g. "Not yet wired into `emu.sv`" tells you the wiring is the next task).
- List **known limitations** explicitly (e.g. `length == 0` PES not handled) so they
  aren't rediscovered the hard way.
- Cross-reference: point from `CLAUDE.md` summaries to the detailed `docs/` section, and
  name the relevant files/modules/testbenches so they're easy to locate.
- Keep `docs/roadmap.md` current — it's the canonical "what's next" across sessions.

### ★ Keep README.md current (mandatory — it is the user-facing contract)

`README.md` states what the core does and doesn't do — "What works", "Known
limitations", supported formats/resolutions, tools, controls, settings, on-screen
messages, acknowledgements. **Whenever a change invalidates or adds to any statement in
the README, update the README in the SAME change.** A README that still lists a shipped
feature as a limitation (or vice versa) misleads every user and evaluator who reads it
— treat it exactly like a stale status marker: a documentation bug, fix on sight.
Concrete triggers: a new codec/format/resolution, a limitation removed or discovered, a
new user-facing tool in `tools/`, new OSD settings or buttons, new on-screen messages,
new external references worth acknowledging. (Instituted 2026-08-24 after the MPEG-1/MP2
feature landed while the README still said "MPEG-1 video is not supported".)

### ★ Update status markers when a feature completes (mandatory — a stale marker is a bug)

The docs went stale once (2026-07-09 reconciliation, PR after fj#93) because status wording
was written at branch-creation time and never updated when the PR merged — so a whole batch
of shipped, HW-confirmed menu work (PR fj#84–fj#90) still read "sim-verified, HW gate pending"
and misdirected a "what's next?" session. To prevent recurrence:

- **When you complete or merge a feature, update its status markers in the SAME change** —
  both `docs/roadmap.md` and any per-feature status header in `docs/` (e.g. section headers
  in `docs/dvd_menu_refinements.md`, `docs/dvd_nav.md`, `docs/dvd_vm.md`).
- Flip the marker to reality: `🔧`/`❌`/`[ ]`/"sim-verified, HW gate pending" →
  `✅ MERGED (PR #NN)` and, once the board test passes, `✅ HW-CONFIRMED`. If merged but not
  yet hardware-tested, say exactly that (`⏳ HW-confirm pending`) — don't leave it reading "gate pending".
- **Retire dead branch names.** A merged feature must not still point at a live `feature/*`
  branch in prose — replace it with the PR number.
- Treat a lingering `🔧`/"HW gate pending"/`feature/*` reference on shipped work as a
  documentation bug: fix it on sight. When in doubt about true status, reconcile against
  `tea pr list --state closed` (what actually merged), not the branch name.

---

## Repository Structure

```
MiSTer_DVD/
├── CLAUDE.md                  ← you are here
├── docs/
│   ├── architecture.md        ← full system design & data flow
│   ├── audio.md               ← AC-3, DTS, LPCM audio strategy
│   ├── roadmap.md             ← phased implementation plan

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owenb321/MiSTer_DVD](https://github.com/owenb321/MiSTer_DVD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
