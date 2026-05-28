---
trigger: always_on
description: Windows (${TGT2_WINDOWS_HOST})                 Mac (${TGT2_MAC_HOST})
---

# T-GT II + Forza Telemetry — Project Notes

## Architecture (Bun + TypeScript)

```text
Windows (${TGT2_WINDOWS_HOST})                 Mac (${TGT2_MAC_HOST})
┌────────────────────────────┐               ┌──────────────────────┐
│ bun run src/server.ts      │   WS 8765     │ bun run src/proxy.ts │
│ ┌─ FFI winmm.dll (30Hz)   │ ──────────▶   │ relay → WS 8766      │
│ ├─ UDP :6688 (Forza)       │               │ dashboard.html       │
│ ├─ Adaptive Auto-Shift     │               │ served on HTTP 9999  │
│ │  └─ power-curve lookup   │               └──────────────────────┘
│ ├─ 60Hz latest-frame fanout│
│ │  ├─ dashboard WS         │
│ │  ├─ overlay WS /overlay  │
│ │  ├─ autoshift            │
│ │  └─ power-curve Worker   │
│ │     └─ shared snapshots  │
│ └─ WPF native overlay      │
│                            │
│ start_key_agent.bat        │
│ ┌─ bun run src/key_agent.ts│
│ ├─ HTTP :7788 (gear/keys)  │
│ ├─ vJoy FFI (DirectInput)  │
│ └─ user32 FFI (fallback)   │
└────────────────────────────┘
```

## Stack

- **Runtime**: Bun 1.3.14 (Windows), Bun 1.3.10 (Mac)
- **Language**: TypeScript (strict mode)
- **Joystick**: `bun:ffi` -> `winmm.dll joyGetPosEx`
- **Telemetry**: `node:dgram` UDP socket
- **WebSocket**: built-in `Bun.serve`
- **Native Overlay**: PowerShell/WPF, launched from the compiled app
- **Gear Control**: `bun:ffi` -> `vJoyInterface.dll`
- **Key Injection**: `bun:ffi` -> `user32.dll SendInput` / `keybd_event`

## Source Files

| File | Purpose |
|------|---------|
| `src/server.ts` | Windows combined server entry point |
| `src/app.ts` | Single-process Windows app entry point |
| `src/overlay.ts` | Starts the Windows native floating overlay |
| `src/windows_overlay.ps1` | PowerShell/WPF overlay UI |
| `src/wheel.ts` | winmm joystick reader |
| `src/forza.ts` | Forza UDP telemetry parser |
| `src/autoshift.ts` | First-principles power-curve auto-shift |
| `src/power_curve_pipeline.ts` | Non-blocking shared snapshot reader / Worker input |
| `src/power_curve_worker.ts` | Per-car 1 RPM curve learning, upper-quantile aggregation, and output smoothing |
| `src/power_curve_types.ts` | Power curve snapshot and worker message contract |
| `src/key_agent.ts` | vJoy/key input agent |
| `src/proxy.ts` | Mac WebSocket relay proxy |
| `dashboard.html` | Browser dashboard |

## Key Agent

Key Agent runs on the interactive desktop session (`start_key_agent.bat`) and listens on `http://0.0.0.0:7788`.

It must be started by double-clicking `start_key_agent.bat` on the Windows desktop. SSH Session 0 cannot inject input into Session 1.

### API endpoints

- `/gear/N` — direct gear selection via vJoy (N = -1 to 10)
- `/clutch` — pulse vJoy button 12 for clutch binding
- `/up`, `/down` — sequential shift
- `/throttle/on|off`, `/brake/on|off` — hold/release keys
- `/method/A|B|E` — switch injection method
- `/ping` — health check

## Adaptive Auto-Shift

The active algorithm does not use a neural network. It is a direct physics lookup:

1. Learn one engine power curve per car/tune from clean high-throttle telemetry.
2. Learn each gear ratio from engine RPM and driven wheel speed.
3. At each frame, convert current wheel speed into RPM for `gear-1`, `gear`, and `gear+1`.
4. Compare estimated horsepower and shift only when the target gear clears shift-cost and anti-oscillation guards.

### Learning filters

- High throttle only (`accel / 255 >= 0.80`)
- Grounded suspension
- Low tire slip
- Clean surface: low rumble strip and puddle depth
- Top-sample pool per RPM bin, using high-percentile torque to avoid weak terrain samples dragging the curve down

### Guards

- Fuel-cut / post-peak usable RPM ceiling
- Fuel-cut / ceiling-triggered upshifts have an independent `500ms` debounce to prevent one limiter event from chaining multiple upshifts.
- Brake blocks upshift
- Airborne protection
- Slip guard, relaxed in low gears
- Minimum shift cooldown
- Post-shift settle window
- Shift execution must wait for three conditions before releasing planning: telemetry target gear, clutch released, and RPM matching wheel speed plus learned gear ratio within tolerance. Keep this wait bounded by timeout.
- All key-agent HTTP calls from auto-shift must use a hard timeout. A stuck `/gear/hold/N` request can otherwise leave `shiftExecutionLocked=true` and stop automatic planning while the process still appears alive.
- Reversal lock
- Larger threshold for downshift advantage
- Directional manual paddle override

## Telemetry Distribution

The server must not queue historical telemetry for dashboard, overlay, or shifting:

1. UDP capture replaces one latest-frame slot as packets arrive.
2. A fixed `TGT2_TELEM_HZ` tick, normally `60`, distributes at most one newly captured frame.
3. The same distributed frame fans out to the dashboard WebSocket, `/overlay` WebSocket, auto-shift consumer, and power-curve Worker.
4. If no new UDP frame arrives, no telemetry frame is distributed. A display still advancing after input stops indicates an old build or client-side backlog.
5. Each outbound frame has a monotonically increasing `seq`; clients can detect skipped or delayed frames without replaying stale data.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yuandiaodiaodiao/forza-horizon-6-tgt2-shifter](https://github.com/Yuandiaodiaodiao/forza-horizon-6-tgt2-shifter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
