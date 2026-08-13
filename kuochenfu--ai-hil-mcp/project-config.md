---
trigger: always_on
description: validates it and shows what each board resolves to.
---

# CLAUDE.md — AI-HIL Embedded Dev Automation

Operating instructions for Claude Code on this repository.
Follow the SOPs below exactly when working with physical hardware.

---

## Project Overview

**AI-HIL (AI-Hardware-in-the-Loop)** lets Claude Code perceive, act on and
validate physical embedded hardware through MCP servers.

Layout:

```
Cargo.toml          workspace root — one target/ for all six servers
hil-core/           shared: device registry, anomaly rules, resource locks,
                    command execution with timeouts, structured tool output
<name>-mcp-rs/      the six MCP servers
devices.toml        single source of truth for every board, camera and mic
orchestrator/       Python closed-loop driver + metrics
doc/known-bugs.md   the Known Bug Record
```

## Active MCP Servers

| Server | Tools |
|--------|-------|
| `serial-mcp` | `list_serial_ports`, `list_boards`, `read_serial_log`, `send_serial_command`, `read_multi_log`, `wait_for_pattern`, `start_logging`, `stop_logging`, `clear_log_buffer` |
| `jtag-mcp` | `list_probes`, `list_boards`, `halt_cpu`, `resume_cpu`, `reset_target`, `read_registers`, `read_memory`, `write_memory`, `read_call_stack`, `diagnose_hardfault`, `set_breakpoint`, `clear_breakpoint`, `set_watchpoint`, `clear_watchpoint`, `read_rtt_log`, `start_rtt_logging`, `stop_rtt_logging` |
| `build-flash-mcp` | `list_projects`, `build_firmware`, `clean_build`, `get_build_size`, `flash_firmware`, `erase_flash` |
| `ppk2-mcp` | `find_ppk2`, `get_metadata`, `measure_current`, `profile_power_states`, `measure_with_pin_trigger`, `estimate_battery_life`, `set_dut_power` |
| `vision-mcp` | `list_cameras`, `list_boards`, `get_camera_info`, `set_resolution`, `set_ptz`, `adjust_image`, `capture_frame`, `start_stream`, `stop_stream`, `grab_frame`, `start_recording`, `stop_recording` |
| `audio-mcp` | `list_audio_devices`, `get_audio_info`, `capture_audio`, `detect_frequency`, `measure_noise_level`, `detect_tone` |

Binaries live in `target/release/` (workspace build). Rebuild with `./setup.sh`.

### How to read tool results

Every tool returns **human text plus a structured JSON body**, and failures set
`isError`. Branch on the JSON, not on the prose:

- `ok: true|false` is present on every result.
- `build_firmware` → `built: true|false`; `flash_firmware` → `flashed: true|false`.
- `diagnose_hardfault` → `cfsr_flags: []`, `hfsr_flags: []`, `in_fault: bool`, `fault_pc`.
- `read_serial_log` / `read_rtt_log` → `lines: []`, `anomaly: { kind, matches }`.
- `capture_frame` / `grab_frame` → an actual image block you can look at.

A failed tool call is an error, not text to interpret. If `build_firmware`
returns `isError`, **do not flash** — fix the build first.

## Target Hardware

Everything is declared in `devices.toml`; `cargo run -p hil-core --example doctor`
validates it and shows what each board resolves to.

- **STM32WL55JC** (NUCLEO-WL55JC1 x4) — sub-GHz LoRa, ultra-low-power, dual-core (CM4 + CM0PLUS)
  - Aliases `stm32a`–`stm32d`, each pinned to its own ST-Link by `probe_serial`
  - Firmware: `/Users/chenfu/Labs/stm_projects/synapse-lora`, preset `Debug`
    (the parent project builds both CM4 and CM0PLUS)
  - `artifacts` in `devices.toml` names the two ELFs explicitly — the tree also
    holds stale copies under `build/`, so never rely on globbing
  - RTT logging is available over SWD (`read_rtt_log`), faster than UART and
    usable while the CPU is blocked
  - **Option bytes:** SBRV must be `0xC000` (CM0+ boots from `0x08030000`).
    Factory default `0x8000` points at erased flash → CM0+ never boots → CM4
    hangs at mailbox sync.
- **ESP32-S3 Gateway** (`board1`) and **Node** (`board2`) — Zenoh relay / sensor node
  - Build with `idf`, flash with `esptool`; log and shell are separate ports
- **Nordic PPK2** — power measurement; auto-detected, or set `ppk2_port` per board
- **Logitech MX Brio Ultra 4K** — visual inspection (`camera.brio`, index 0)

---

## Safety Constraints

Some of these are now enforced in code; the rest still depend on you.

**Enforced by the servers:**
- Supply voltage is capped at each board's `max_voltage_mv` (default 3600 mV).
  `ppk2-mcp` refuses anything higher instead of driving it into the DUT.
- `erase_flash` requires `confirm=true`.
- Every probe/port operation takes a cross-process lock, so a flash cannot
  collide with a live debug session or a log collector. A contended call fails
  with the name of the holder rather than corrupting the session.
- External commands (OpenOCD, idf.py, esptool, cmake) run under a timeout.

**Your responsibility:**
- Never modify an ISR without reading the call stack first (`read_call_stack`).
- Never flash if the build returned `isError`.
- Wait 3 s after flashing before reading the log — the board needs to boot.
- Watchdog timeout is 2 s: do not leave the CPU halted for more than ~1.5 s
  during live diagnosis.
- If `diagnose_hardfault` shows `FORCED` in `hfsr_flags`, always read
  `cfsr_flags` for the real cause before touching code.
- ESP32: `resume_cpu` may fail after a halt on Xtensa — use `reset_target`.

### Who owns the hardware

Only one thing can hold a probe or a port at a time:

| Before this | Do this |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kuochenfu/ai-hil-mcp](https://github.com/kuochenfu/ai-hil-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
