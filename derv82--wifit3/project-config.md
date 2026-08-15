---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Cheatsheet

- **Git**: Commit directly to `master`; branch, switch branches, or make worktrees only when asked. The working tree is shared across concurrent sessions: `git status` may show files, and tests may fail, from work that isn't yours; stage only your task's files (`git add <paths>`, never `-A`/`.`).
- **Comments / code style**: a small closed allowlist: a concise docstring (only where the name doesn't carry it), citations/magic-value notes, phase landmarks; everything else is noise. When unsure, omit; prefer naming over commenting. Full rules in `docs/porting/CODE-STYLE.md`.
- **Cross-platform by design**: Wifit3 uses PyUSB + `libusb_package` so drivers run on Windows (with Zadig binding the device to WinUSB) AND Linux (after `rmmod <kernel_driver>`). No Kali boot is needed for normal dev. That's the whole point of going userland.
- **Hardware testing: the agent runs it.** Offline pcap verification (`scripts/porting/verify_pcap.py <chip>`) tests the port against the recorded wire without hardware, so chipset bring-up is now agent-driven: once the user has plugged in + WinUSB-bound (Zadig) the card, the agent runs `python scripts/<chipset>/test_hw.py` (optionally `--debug`) **itself**, pcap-verifies + HW-smoke-tests each milestone, and commits each: no per-iteration handoff. Everything including wiring *and* firing TX (channel tune, 2.4/5 GHz RX, TX-descriptor build) is the agent's to complete autonomously. The `beacon_watch.py` (live) vs `beacon_watch_usbcap.py` (the `driver_captures/` capture's beacon count) A/B is the RX-health check.
- **Device gets borked? User replugs.** That resets cold-boot state. You can suggest "please unplug, wait a few seconds, replug, then rerun" if a previous attempt left it stuck.
- **Porting / bringing up a chip?** The playbook lives in `docs/porting/METHODOLOGY.md` (or run `/port <chip>`): port from the C source, verify each milestone against the cold-boot pcap, commit each. Run the loop to a stopping point yourself: surface only for a decision you can't make from source+pcap, the live-TX gate, a committed milestone, or a real block. Don't narrate progress.
- **Register READs can mutate device state: never assume two reads commute, never reorder them vs the capture.** Read-to-clear status regs, latch-on-read pairs, FIFO pops, indirect-access auto-advance: `READ X; READ Y` ≠ `READ Y; READ X` on silicon, and out-of-order reads strand the card in a state the capture never visited. So the verify tool's strict-positional cursor (reads included) is a *correctness* gate, not pedantry: a reordered-read divergence is a real driver bug to fix, never a tolerance to add.
- **Per-chipset port-reference docs**: each chip dir has a `<CHIP>.md`: a short README for the chip (status, gotchas, orientation, scripts, and a debug log for open/unresolved findings). Keep it to what isn't already in the code. The debug log is for a live investigation and what's been ruled out, never a port-order/milestone log (that history is in git). Template + rules in `docs/porting/CHIP-DOC.md`.
- **Human-facing docs are the face of the project.** `README.md` + `docs/SUPPORTED-HARDWARE.md`: edit only when the user asks, and show the proposed edit for approval before writing (terse, observational, no port-accuracy braggadocio; that belongs in `<CHIP>.md` + commits). Prefer prose direction over multiple-choice for these.
- **Within `chips/`, don't re-use code from another driver.** *Why:* a shared core meant a fix for one card forced re-testing every card and risked regressing the others.
- **Lead's rule**: discuss class design (`Driver` vs `WlanInterface` responsibilities, etc.) BEFORE execution. Treat the user as Senior Lead.
- **Never write to auto-memory without asking.** Before saving or updating any file under the auto-memory dir (`MEMORY.md` + its entries), show the user the proposed entry and wait for explicit approval. This overrides the default proactive-save behavior. The user owns what goes into always-loaded context.
- **Planning docs** (NOT auto-loaded, open as needed): `docs/planning/FEATURES.md` (capabilities to build), `docs/planning/BUGS.md` (defects + QoL to fix). Current per-card state: `docs/SUPPORTED-HARDWARE.md` (grading process: `docs/GRADING.md`). Porting playbook: `docs/porting/` (or `/port <chip>`).

## Commands

This repo uses **`uv`** for env management. The system `python` on PATH does NOT have project deps: always run Python via `uv run` (or `.venv\Scripts\python.exe`). Quick import probes like `python -c "import textual"` from the agent will fail with `ModuleNotFoundError`. Use `uv run python -c "..."` instead.

```bash
# Install (editable, with dev deps)
uv sync --group dev               # preferred; or: pip install -e ".[dev]"

# Run
uv run wifit3                     # or: uv run python -m wifit3

# Tests
uv run pytest                          # all tests
uv run pytest tests/chips/ar9271_v2/   # single module
uv run pytest tests/wlan/test_parser.py::test_wlan_frame_parser_extracts_ssid

# Lint (lint only: NEVER format)
uv run ruff check src/

# Textual live dev (hot-reload)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derv82/wifit3](https://github.com/derv82/wifit3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
