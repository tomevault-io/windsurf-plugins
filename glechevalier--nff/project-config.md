---
trigger: always_on
description: - Always use `nff flash --sim` to compile — never call arduino-cli directly.
---

# nff — Wokwi Simulation Context

## Hard Rules
- Always use `nff flash --sim` to compile — never call arduino-cli directly.
- Always use `nff wokwi run` or `nff wokwi run --gui` to simulate.
- Never install libraries with arduino-cli. Use built-in ESP32 APIs only,
  or ask the user to install the library first.
- For ESP32 servo/PWM use ledcAttach + ledcWrite (built-in LEDC, no library).

## Project
- Board : arduino:avr:uno
- FQBN  : arduino:avr:uno
- Chip  : wokwi-arduino-uno

---

## Simulation Pipeline

```
1. Write sketch      sketches/<name>/<name>.ino
2. Edit circuit      diagram.json  (add components + wiring)
3. Compile           nff flash --sim sketches/<name> --board arduino:avr:uno
4. Visual sim        nff wokwi run --gui
   Headless sim      nff wokwi run [--timeout MS] [--serial-log FILE]
5. Fix bugs, repeat from step 3
```

wokwi.toml must point to the compiled ELF:
  firmware = "sketches/<name>/build/arduino.avr.uno/<name>.elf/<name>.ino.elf"

---

## diagram.json — Component Wiring

Always wire the serial monitor:
  ["esp:TX0", "$serialMonitor:RX", "", []]
  ["esp:RX0", "$serialMonitor:TX", "", []]

ESP32 pin names: esp:D<gpio>  esp:GND.1  esp:GND.2  esp:3V3  esp:VIN

Common components:
  wokwi-led          attrs: color (red/green/blue/yellow)
  wokwi-pushbutton   attrs: color — pins: btn:1.l (gpio side), btn:2.l (GND side)
  wokwi-servo        attrs: minAngle "-90", maxAngle "90" — pins: PWM, V+, GND
  wokwi-resistor     attrs: value (ohms)

Pushbutton wiring (with INPUT_PULLUP in sketch):
  ["esp:D15", "btn1:1.l", "green", []]
  ["esp:GND.2", "btn1:2.l", "black", []]

Servo connection:
  ["esp:D18",  "srv1:PWM", "orange", []]
  ["esp:3V3",  "srv1:V+",  "red",    []]
  ["esp:GND.1","srv1:GND", "black",  []]

---

## ESP32 Servo — LEDC (no library required)

Wokwi servo maps its full range to 500 µs – 2500 µs.
50 Hz / 16-bit resolution (max count 65 535, period 20 000 µs):

  −90°  →  duty 1638   (500 µs)
    0°  →  duty 4915  (1500 µs)
  +90°  →  duty 8192  (2500 µs)

```cpp
ledcAttach(SERVO_PIN, 50, 16);   // ESP32 Arduino core 3.x
ledcWrite(SERVO_PIN, 4915);      // center
```

Always set minAngle: "-90" and maxAngle: "90" on wokwi-servo in diagram.json.

---

## Debugging

- Compile error     → fix sketch, re-run nff flash --sim
- Wrong output      → nff wokwi run --serial-log out.txt, inspect out.txt
- Component silent  → check diagram.json pin names and connection direction
- Servo wrong angle → verify duty values match the 500–2500 µs Wokwi range
- Button not firing → INPUT_PULLUP + wiring gpio→btn:1.l, GND→btn:2.l

---

# nff — Rust Architecture

## Status

The Python `nff` binary has been fully replaced by a compiled Rust binary — single executable,
no Python runtime required for end users, stronger types, better cross-platform packaging.

The MCP server is now native Rust (`nff-rs/nff/src/mcp_server.rs`, rmcp crate). Wokwi
integration is also native Rust (`tools/wokwi.rs`). Only `nff test` still delegates to the
Python package via subprocess.

**Adding a new MCP tool:** add it to `nff-rs/nff/src/mcp_server.rs` — implement a method on
`NffServer` with the `#[tool(...)]` attribute. Do not edit `nff/nff/mcp_server.py`; it is
superseded.

## Migration Scope

**IN (rewrite in Rust):**
- CLI entry point and all command definitions
- `nff/nff/config.py` → `tools/config.rs`
- `nff/nff/tools/serial.py` → `tools/serial.rs`
- `nff/nff/tools/boards.py` → `tools/boards.rs`
- `nff/nff/tools/toolchain.py` → `tools/toolchain.rs`
- `nff/nff/tools/installer.py` → `tools/installer.rs`
- All commands: `flash`, `init`, `monitor`, `doctor`, `clean`, `connect`, `ota`, `install-deps`
- `commands/mcp.rs` (calls `mcp_server::run()` — native Rust MCP server, no Python)

**OUT (keep in Python):**
- `nff test` command only — delegates to `python -m nff test` via subprocess

## Rust Project Layout

Create at `nff/nff-rs/` (sibling to `nff/nff/`):

```
nff/nff-rs/
├── Cargo.toml              (workspace root)
└── nff/
    ├── Cargo.toml
    └── src/
        ├── main.rs
        ├── cli.rs
        ├── commands/
        │   ├── mod.rs
        │   ├── flash.rs        (replaces commands/flash.py)
        │   ├── init.rs         (replaces commands/init.py)
        │   ├── monitor.rs      (replaces commands/monitor.py)
        │   ├── doctor.rs       (replaces commands/doctor.py)
        │   ├── clean.rs        (replaces commands/clean.py)
        │   ├── connect.rs      (replaces commands/connect.py)
        │   ├── ota.rs          (replaces commands/ota.py)
        │   ├── install_deps.rs (replaces install-deps command in cli.py)
        │   └── mcp.rs          (spawns Python MCP server)
        └── tools/
            ├── mod.rs
            ├── config.rs       (replaces config.py)
            ├── serial.rs       (replaces tools/serial.py)
            ├── boards.rs       (replaces tools/boards.py)
            ├── toolchain.rs    (replaces tools/toolchain.py)
            └── installer.rs    (replaces tools/installer.py)
```

Workspace `Cargo.toml`:
```toml
[workspace]
members = ["nff"]
resolver = "2"
```

## Cargo.toml Dependencies

```toml
[package]
name = "nff"
version = "0.2.16"   # keep in sync with pyproject.toml
edition = "2021"

[[bin]]
name = "nff"
path = "src/main.rs"

[dependencies]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GLechevalier/nff](https://github.com/GLechevalier/nff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
