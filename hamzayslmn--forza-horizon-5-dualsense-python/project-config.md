---
trigger: always_on
description: A short tour of the project so you can read, run, and modify it in an afternoon.
---

# AGENTS.md — Onboarding for New Developers

A short tour of the project so you can read, run, and modify it in an afternoon.

---

## 1. What is this project?

A small Python service that gives the **PlayStation DualSense controller real
adaptive‑trigger feedback while playing Forza Horizon 5 on PC (Steam)**.

- Forza Horizon 5 broadcasts live telemetry (RPM, speed, pedals, tire slip, gear…)
  over **UDP** if you turn on **HUD & Gameplay → Data Out** in the game.
- Steam Input only sends generic rumble to the DualSense; the trigger motors
  do nothing.
- This project listens to the UDP feed, computes a trigger force/vibration
  every frame, and writes it directly to the controller via **raw HID**, while
  carefully **not touching the rumble bytes** so Steam still drives rumble.

Result: brake trigger that resists like a brake pedal, throttle trigger that
pushes back when the engine works, ABS pulse on tire slip, gear‑shift thump,
rev‑limiter buzz.

---

## 2. Tech stack

| Piece | What |
|---|---|
| Language | Python (project requires `>=3.14`, see `src/pyproject.toml`) |
| Package manager | [`uv`](https://astral.sh/uv) (fast, replaces pip + venv) |
| Single dependency | [`hidapi`](https://pypi.org/project/hidapi/) for raw HID I/O |
| OS | Windows (HID layer + `start.bat` are Windows‑specific) |
| Hardware | DualSense or DualSense Edge over USB or Bluetooth |

No tests, no CI, no linter run automatically. `ruff` line length is set in
`pyproject.toml` but not enforced.

---

## 3. Repository layout

```
Forza-Horizon-5-DualSense-Python/
├── README.md            # User‑facing docs (install, in‑game setup, tuning)
├── AGENTS.md            # ← you are here (developer onboarding)
├── LICENSE
├── start.bat            # Windows launcher: installs uv if missing, runs main.py
├── img/                 # Screenshots used in README
├── .github/FUNDING.yml  # GitHub sponsorship config
└── src/
    ├── pyproject.toml   # Project metadata + dependencies
    ├── uv.lock          # Locked dependency versions (do not edit by hand)
    ├── main.py          # Entry point: arg parsing + main packet loop
    └── modules/
        ├── __init__.py        # Exposes setup_logging() + sub‑packages
        ├── settings.py        # 👈 ALL tunables live here (one dataclass)
        ├── dualsense/
        │   ├── __init__.py    # Re‑exports DualSense, TriggerAnimation, triggers
        │   ├── main.py        # HID layer (open/close/write to controller)
        │   └── triggers.py    # Effect primitives + per‑frame TriggerAnimation
        └── udplistener/
            ├── __init__.py    # Re‑exports UDPListener, parse_packet
            └── main.py        # UDP socket + 324‑byte FH5 packet parser
```

The whole codebase is **~6 small files**. Read them in this order:

1. `src/main.py`
2. `src/modules/settings.py`
3. `src/modules/udplistener/main.py`
4. `src/modules/dualsense/triggers.py`
5. `src/modules/dualsense/main.py`

---

## 4. How the data flows (one frame)

```
Forza Horizon 5  ──UDP 5300, 324 bytes──►  UDPListener.recv_latest()
                                                  │
                                                  ▼
                                          parse_packet(pkt) -> dict
                                                  │
                                                  ▼
                                  TriggerAnimation.update(t, settings)
                                                  │
                                                  ▼
                                       (left, right) trigger commands
                                                  │
                                                  ▼
                                            DualSense.set(left, right)
                                                  │  (worker thread writes HID
                                                  │   only when state changes)
                                                  ▼
                                          DualSense controller motors
```

Each trigger command is a 3‑tuple `(mode, p1, p2)`:
- `M_OFF (0x05)` — trigger free
- `M_RIGID (0x01)` — constant resistance, p2 = force 0..255
- `M_PULSE (0x06)` — vibration, p1 = freq Hz, p2 = amplitude 0..255

The HID write only flips `valid_flag0` bits for the trigger motors, so Steam
Input keeps owning the rumble bytes.

---

## 5. The five files in detail

### `src/main.py` — entry point
- Parses CLI args (`--host`, `--port`, `--debug`, optional trailing `game_cmd`).
- Builds a `Settings()` and overrides host/port from CLI.
- Calls `run(settings, game_cmd)` which:
  1. Opens the `DualSense` (HID write thread + optional startup pulse).
  2. Opens a `UDPListener` context manager.
  3. Enters `_loop`: pull latest packet, parse, compute trigger output,
     write only when it changes, log a debug line once per second.
- If `game_cmd` is passed (Steam wrapper mode), spawns the game as a child
  process and exits when the game exits.

### `src/modules/__init__.py` — logging
- One helper: `setup_logging(debug)`. ANSI‑colored output on Windows, level
  switches between INFO and DEBUG.

### `src/modules/settings.py` — the only file most users edit
- A single `@dataclass Settings` with **flat fields** (no presets, no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HamzaYslmn/Forza-Horizon-5-DualSense-Python](https://github.com/HamzaYslmn/Forza-Horizon-5-DualSense-Python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
