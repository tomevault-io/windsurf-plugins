---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Doctrine — read `DOCTRINE.md`

The rules are below, one line each, because this file is loaded into every session
automatically. **`DOCTRINE.md` carries the reasoning, the history, and the rules that
have been retired** — read it before changing a rule or arguing with one. Doctrine used
to be stated here in full, in the board's GOVERNING section and in the spec files at
once; three copies drifted, and five blocks sat here marked ÜBERHOLT while still being
loaded into every context window.

**Binding:**

1. **One runtime, and it is a separate process.** C64RE has no emulator of its own —
   no in-process machine, no fallback, no second implementation to A/B against. If the
   runtime is unavailable, tools say so and carry the setup recipe; they never quietly
   do something else. Do not reintroduce an in-repo core. (Spec 806; the single-path
   rule it replaces is retired in `DOCTRINE.md`.)
2. **One SHARED machine, plus ephemeral sandboxes.** The session the human sees is
   exactly one, and the UI shows only that one: human and LLM co-drive it, and it is
   never power-cycled for a test. For point work — depacking, a test run, a comparison
   — C64RE **may** spawn its own `trx64-daemon` sandbox on its own port. A sandbox is
   born with a budget and **ends itself** when the budget runs out, whether or not
   anyone is still listening. Amended 2026-08-14; the reasoning and the reaping rule
   are in `DOCTRINE.md`.
3. **The 1541 drive CPU stays its own 6502** (a TRX64 rule now). Do not merge it into
   the C64 core.
4. **Traces go into the trace store, never into a one-off script.** Capture the whole
   window and filter on query. `console.log` is a debug primitive, not a trace.
5. **Read before you hypothesise.** Read the reference end-to-end first and say so.
   Report the FIRST divergence and its window — not statistics, not hotspots. Suspect
   the conversion before the algorithm.
6. **API first.** MCP tool / library / endpoint with smoke coverage, then UI. Never UI
   without the API underneath.
7. **Spec-driven.** `PLAN.md` → a spec under `specs/` → implementation. Spec numbers are
   **shared across C64RE and TRX64**; `specs/README.md` is the one registry and the next
   free number comes from there.
8. **Inside an RE project** (a `C64RE_PROJECT_DIR` workspace, not this repo): load
   `docs/agent-doctrine.md`, run `agent_onboard`, persist with `agent_record_step` and
   the `save_*` family, and search before re-deriving. Roles via `agent_set_role`. These
   do **not** apply to ordinary edits in this repo.

9. **Docs are brought in line BEFORE the commit — at the very latest before the push.**
   Not "later", not "when we tidy up". Before you type `git commit`, every document the
   change touched is already correct: the spec's own `**Status:**` line, its row in
   `specs/README.md`, and any doc that asserted the old state. Close what is finished and
   move it to `_archive/` with its decision. If a push is imminent that is the hard
   deadline — nothing leaves this machine describing a world that no longer exists.
   This is the rule the repo learned the hard way: 726 was carried as open work for ten
   weeks after the spec itself recorded it shipped, 794–798 said PROPOSED while the board
   said DONE, and 784 still says "ready for build" with all its deliverables sitting in
   `src/`. Nine specs were closed in one evening and **not one needed building**.

**Retired — do not re-apply:** VICE and the TypeScript runtime as *authority*
(2026-07-15), and then as anything at all (2026-08-12, Spec 806): the TS emulator, its
1541, its test surface and the 49 `vice_*` MCP tools are deleted. TRX64 is the runtime.
VICE survives only as a source tree to READ when porting. Regression protection is
TRX64's own gates (Spec 783), not an oracle comparison. The techniques from that era
survive as rules 4 and 5 above. Full text and reasoning: `DOCTRINE.md`.

**Ownership.** This agent owns and stewards **both** repos — C64RE and the sibling
TRX64 (`../TRX64`). Leitregel: capability → TRX64, meaning and memory → C64RE. That is a
division of one owner's work, not a handoff: when a capability moves "→ TRX64",
delivering it there is in scope. Cross-repo edits and commits in both are normal.

## Project Overview

MCP server for LLM-powered Commodore 64 reverse engineering. Bundles the TRXDis analysis pipeline to provide heuristic disassembly, semantic annotation, and dual-assembler output (KickAssembler + 64tass) for C64 PRG files, disk images (D64/G64), and CRT cartridges.

C64RE is the reverse-engineering workbench, not the emulator: the runtime is TRX64 (Spec 771), reached as a separate daemon process. There is no runtime in this repo — the TypeScript emulator and the VICE bridge were deleted 2026-08-12 (Spec 806). Leitregel: Capability → TRX64, Meaning/Memory → C64RE.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jondalar/C64ReverseEngineeringMCP](https://github.com/Jondalar/C64ReverseEngineeringMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
