---
trigger: always_on
description: > LLM-native debug probe and firmware development platform — Rust CLI, Python MCP server, CMSIS-DAP, variable streaming
---

# Embedded Agent Bridge (EAB)

> LLM-native debug probe and firmware development platform — Rust CLI, Python MCP server, CMSIS-DAP, variable streaming

Background daemons bridging AI coding agents to debuggers and embedded hardware (ESP32, STM32, nRF, NXP MCX via serial, RTT, J-Link, OpenOCD). **ALWAYS use eabctl for ALL hardware operations.**

## Rules

1. **Execute, don't describe.** Do the work. Never describe what you would do — do it.
2. **Verify from ground truth.** Check the file, run the command, read the output. Never trust memory.
3. **Capture discoveries.** Write learnings to `agent_reports/`. Update knowledge docs after every session.
4. **Run verify gate before declaring done.** Paste green output from `scripts/verify-affected.sh`. "Looks good" is not a gate.
5. **Default to cheapest model that works.** Haiku handles 80% of tasks. Escalate only when quality demands it.
6. **NEVER use esptool/JLinkExe/openocd directly** — Use `eabctl` instead. Direct tool access bypasses EAB's port management and causes "port busy" errors.
7. **NEVER use pio device monitor** — Use `eabctl tail` instead. Direct serial access conflicts with EAB daemon.
8. **NEVER access serial ports or debug probes directly** — EAB manages all hardware interfaces. Direct access causes resource conflicts.
9. **Port busy errors?** Run `eabctl flash` — it handles port release automatically.
10. **Before flashing, check `docs/usb-port-mapping.md`** — Ports shift on USB re-enumeration.
11. **ESP32 multi-probe: ALWAYS use `adapter serial`** — ESP32-C6 and P4 share VID:PID `303a:1001`.
12. **ST-Link V3 invisible on macOS** — See `docs/macos-flash-troubleshooting.md` for workarounds.
13. **Hardware safety: debug probes can brick devices.** Always run `eabctl preflight` before flashing. See `docs/SEVERITY.md` for triage.

## Dispatch

- `/sh <task>` — Haiku agent (config edits, single-file changes, docs, formatting)
- `/ss <task>` — Sonnet agent (features, multi-file changes, refactors, code review)
- `/so <task>` — Opus agent (architecture decisions, cross-cutting reasoning — expensive, use sparingly)

## Agents

| Agent | Domain | Model |
|-------|--------|-------|
| eab-dev | Python/Rust implementation, CLI commands, daemon changes | sonnet |

## Quick Reference

| Need | Location |
|------|----------|
| System reality | docs/SYSTEM.md |
| Knowledge index | docs/INDEX.md |
| Severity rubric | docs/SEVERITY.md |
| Agent guide | AGENT_GUIDE.md |
| Debugging guide | DEBUGGING.md |
| USB port mapping | docs/usb-port-mapping.md |
| macOS flash troubleshooting | docs/macos-flash-troubleshooting.md |
| MCP setup | docs/mcp-setup.md |
| Feature roadmap | docs/FEATURE_ROADMAP.md |
| Agent handoff | .agent/handoff.md |
| Agent reports | agent_reports/ |

## Session Start

1. Read `docs/SYSTEM.md` — what's live vs dormant vs planned
2. `git log --oneline -5` — what was last done
3. `ls -lt agent_reports/ | head -5` — pending results
4. Report: done / in-progress / blockers. Wait for direction.

## Session End

1. Update `docs/SYSTEM.md` "Current State" section — what changed, next steps
2. Write learnings to `agent_reports/YYYY-MM-DD-session.md`
3. Commit `.claude/`, `docs/` to main

## eabctl Command Reference

```bash
# Check status (ALWAYS do this first)
eabctl status
eabctl status --json   # machine-parseable

# View serial output
eabctl tail 50
eabctl tail 50 --json  # machine-parseable

# Send command to device
eabctl send "i"

# Flash firmware (handles EVERYTHING automatically)
eabctl flash /path/to/project

# Reset device
eabctl reset

# Fault analysis (Cortex-M crash decode)
eabctl fault-analyze --device NRF5340_XXAA_APP --json
eabctl fault-analyze --device MCXN947 --probe openocd --chip mcxn947 --json

# DWT profiling (function/region performance measurement)
eabctl profile-function --function main --device NRF5340_XXAA_APP --elf build/zephyr/zephyr.elf
eabctl profile-region --start 0x1000 --end 0x1100 --device NRF5340_XXAA_APP
eabctl dwt-status --device NRF5340_XXAA_APP --json

# RTT (Real-Time Transfer) streaming
# J-Link transport (subprocess-based, background logging)
eabctl rtt start --device NRF5340_XXAA_APP --transport jlink
eabctl rtt stop
eabctl rtt status --json
eabctl rtt tail 100

# probe-rs transport (native Rust extension, all probe types)
# Supports ST-Link, CMSIS-DAP, J-Link, ESP USB-JTAG
eabctl rtt start --device STM32L476RG --transport probe-rs
eabctl rtt start --device STM32L476RG --transport probe-rs --probe-selector "0483:374b"

# Note: probe-rs transport does not yet support background logging daemon
# Use for testing connectivity and firmware RTT setup verification
```

## Flashing Firmware

**ONLY use eabctl flash:**

```bash
# Flash ESP-IDF project (auto-detects chip, pauses daemon, flashes, resumes)
eabctl flash /path/to/esp-idf-project

# Erase flash first if corrupted
eabctl erase
eabctl flash /path/to/project
```

The flash command:
1. Automatically pauses daemon and releases the serial port
2. Detects chip type from build config
3. For ESP32 USB-JTAG ports: uses **OpenOCD JTAG** (not esptool) — much more reliable
4. Flashes bootloader, partition table, and app
5. Resumes daemon and shows boot output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shanemmattner/embedded-agent-bridge](https://github.com/shanemmattner/embedded-agent-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
