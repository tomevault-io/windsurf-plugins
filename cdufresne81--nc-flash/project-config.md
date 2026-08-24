---
trigger: always_on
description: We work for an hospital and our work is critical, failure to succeed will result in the lost of live, failure is not an option.
---

# Claude Code Instructions

# PRIME DIRECTIVE
We work for an hospital and our work is critical, failure to succeed will result in the lost of live, failure is not an option.

## General Rules

- **"Question:" prefix** - If a prompt starts with "Question:", answer only. Take no actions (no file edits, no commands).
- **No auto-commit** - NEVER run `git commit` or `git push` unless the user explicitly asks OR requests to "land the plane" (session completion).
- **Incremental notes** - After completing code changes that add, update, or delete functionality, immediately update the "Recent Completed Work" section in `.claude/notes.md`. Only note meaningful changes (new features, behavior changes, significant fixes). Skip trivial changes (typos, formatting, minor refactors). Always check existing entries to avoid duplicates.
- **Changelog** - `CHANGELOG.md` MUST be updated before every commit. Add entries to the `## [Unreleased]` section using Added/Changed/Fixed/Removed subsections. When a version is tagged, the unreleased section becomes the GitHub release notes. Ensure version sections match actual git tags — never leave released work under Unreleased.
- **NEVER commit or push** - Unless the user ask to land the plane or explicitely ask for it.
- **Test coverage** - New features or changes to existing features must be tested. Create tests if none exist AND the behavior is logical and important to verify. Do not write tests for trivial or cosmetic changes.

## Architecture Rules (enforced; rationale in `docs/internal/ARCHITECTURE.md`)

Layering: utils ← core ← {ecu, ui, api, mcp}; main.py composes ui.
- NEVER import `src.ui` from `src.core` / `src.ecu` / `src.utils` / `src.api` / `src.mcp`.
- NEVER import `src.ecu` from `src.core` / `src.utils`.
- State has ONE owner. NEVER share a mutable dict/list between objects for mutation; the owner exposes methods + Qt signals.
- NEVER add a mixin. New window-scale behavior = an owned collaborator object.
- ONE pipeline copy. Table rendering, flash-image prep, and HTTP/socket read loops each live in exactly one module. Extend it; never copy it.
- Signals carry their own context (rom_path, …). NEVER recover context via `sender()`/`parent()` walks. Max 2 signal hops.
- Styling: colors/fonts/QSS come from the theme module (`src/ui/theme.py`). No new inline hex literals in widgets.
- `src/ecu/` is brick-critical: behavior-preserving changes only, unless a hardware test per `docs/internal/WICAN_MANUAL_TEST.md` is run.
- `tests/test_architecture.py` enforces the import rules — keep it green.

## Session Notes

Check `.claude/notes.md` at the start of each session for:
- Pending tasks from previous sessions
- Important context and decisions

Update this file when ending a session with any important notes for next time.

## Key Documentation

Reference these before modifying related functionality:
- `docs/internal/ARCHITECTURE.md` - Layer map + the architecture rules with the incidents that motivated each; read before adding a cross-layer import, a mixin, a shared-state dict, or a duplicated pipeline. Enforced by `tests/test_architecture.py`
- `docs/internal/LOGGING.md` - Logging configuration and exception hierarchy
- `docs/internal/ROM_DEFINITION_FORMAT.md` - XML format for ROM definitions
- `docs/internal/UI_TESTING.md` - GUI test runner, screenshots, and test scripts
- `docs/internal/CODE_AUDIT.md` - Full codebase audit findings (bugs, dead code, duplication, test gaps)
- `docs/internal/WICAN_TRANSPORT.md` - Design & build plan for WiCAN PRO wireless (WiFi/SLCAN) ECU transport; reference before touching the ECU transport/session/flash-connect layer
- `docs/internal/WICAN_MANUAL_TEST.md` - Hardware-in-the-loop checklist for the WiCAN ROM read path (firmware version ping, bench-tool read + byte-compare, UI flow); run after touching the transport, firmware, or adapter-selector UI
- `docs/internal/WICAN_PART_C_FINDINGS.md` - Investigation findings (CAN-wedge reboot root cause + clean-teardown fix, no-reboot protocol switch, unified read+write SD architecture); reference before implementing the firmware reboot fix or deciding the WiCAN WRITE-over-SD architecture
- `docs/internal/WICAN_SLCAN_COEXISTENCE_PLAN.md` - Sequencing plan to replace the protocol-switch reboot with an always-on dedicated SLCAN port that coexists with the datalogger (FLASH_ACTIVE_BIT single-CAN interlock, FWD→FWB merge order, RPM-gated datalog/flash); reference before merging the datalogger firmware branch or building the no-reboot SLCAN port
- `docs/internal/WICAN_DEADMAN_AUTORESUME.md` - Validated design for brick-safe datalog auto-resume when NC-Flash vanishes (lid close / crash / Wi-Fi drop): the HOST_BUS_CLAIM_BIT auth-window fence + firmware dead-man reaper, plus the missing #36 RX-forward fix. Reference before touching datalog pause/resume, the `/datalog` endpoint, the FLASH_ACTIVE_BIT/DATALOG_PARK_BIT interlock, or the host flash auth window

**Rule:** When creating new documentation in `docs/`, add it to this list with a brief description of when to reference it.

## UI Testing & Screenshots

**Tool:** `tools/test_runner.py` - Automated GUI testing with screenshot capabilities

**When to use:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdufresne81/nc-flash](https://github.com/cdufresne81/nc-flash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
