---
trigger: always_on
description: Reusable Lua widgets for EdgeTX colour-screen radios (TX16, Jumper T15, TX15) displaying telemetry, model info, and simulator utilities. Target platform: embedded hardware with limited CPU/memory running Lua 5.1.
---

# EdgeTX Widgets Development Guide

## Project Overview

Reusable Lua widgets for EdgeTX colour-screen radios (TX16, Jumper T15, TX15) displaying telemetry, model info, and simulator utilities. Target platform: embedded hardware with limited CPU/memory running Lua 5.1.

**Domain Context:**
- Target platform: EdgeTX colour radios running Lua widgets in the main/view pages
- Data sources: OpenTX/EdgeTX telemetry sensors (GPS, RSSI, battery, timers), model metadata, and companion simulators for offline testing
- Interaction model: Widgets render via `lcd` APIs within a zone; options are set through EdgeTX widget settings and refreshed each frame
- Constraints: Runs on embedded hardware with limited CPU/memory; avoid heavy allocations and prefer integer math where possible
- Testing: Lua test harness and telemetry simulator under `tests/` mirror on-radio behaviour for CI-style checks

## Agent Conduct

- **Verify assumptions** before executing commands; call out uncertainties first
- **Ask for clarification** when the request is ambiguous, destructive, or risky
- **Summarise intent** before performing multi-step fixes so the user can redirect early
- **Cite the source** when using documentation; quote exact lines instead of paraphrasing from memory
- **Break work into incremental steps** and confirm each step with the smallest relevant check before moving on

## Project Structure

```
edgetx-widgets/
├── widgets/           # Widget implementations
│   ├── BattWidget/    # Battery voltage and current display
│   ├── Dashboard/     # Full-screen comprehensive telemetry view
│   ├── GPSWidget/     # GPS satellite and coordinate display
│   ├── RXWidget/      # Receiver signal strength display
│   ├── SimWidget/     # Simulator model testing utilities
│   ├── TeleView/      # Compact telemetry display
│   └── common/        # REQUIRED: Shared utilities and assets
│       ├── utils.lua  # Typography, battery helpers, time formatting
│       └── icons/     # Shared icons (battery, connection, GPS)
├── tests/
│   ├── lua/           # 63 widget tests (vanilla Lua 5.1)
│   │   ├── test_battwidget.lua    # 12 tests
│   │   ├── test_dashboard.lua     # 11 tests
│   │   ├── test_teleview.lua      # 16 tests
│   │   ├── test_gpswidget.lua     # 6 tests
│   │   ├── test_rxwidget.lua      # 5 tests
│   │   ├── test_simwidget.lua     # 9 tests
│   │   ├── test_simmodel.lua      # 4 tests
│   │   └── setup.lua              # Lua path configuration
│   ├── utils/
│   │   └── telemetry_simulator.lua  # Mock EdgeTX telemetry
│   ├── run_tests.bat  # Windows test runner
│   └── run_tests.sh   # Linux/Mac test runner
└── docs/
    ├── TELEMETRY_GUIDE.md         # Sensor reference
    └── COMPANION_SIMULATOR_GUIDE.md  # Integration guide
```

## Architecture

### Widget Structure Pattern

Every widget follows this module return pattern in `main.lua`:

```lua
-- Load shared utilities (REQUIRED)
local utils = loadScript("/WIDGETS/common/utils.lua")()

local function create(zone, options)
  -- Preload icons/resources once during creation
  return { zone = zone, cfg = options, icons = {...} }
end

local function update(widget, options)
  widget.cfg = options
end

local function refresh(widget, event, touchState)
  -- Render frame using lcd.drawText() and lcd.drawBitmap()
  -- Use utils.text() for consistent typography
end

local function destroy(widget)
  -- CRITICAL: Clean up bitmaps to prevent memory leaks
  for _, bmp in pairs(widget.icons or {}) do
    if bmp and bmp.delete then bmp:delete() end
  end
end

return { name = "WidgetName", options = {...}, create = create, update = update, refresh = refresh, destroy = destroy }
```

### Required Dependencies

- **`widgets/common/utils.lua`**: Shared utilities for typography, battery calculations, time formatting. Load via `loadScript("/WIDGETS/common/utils.lua")()`
- **`widgets/common/icons/`**: Shared icon assets (battery, connection, GPS). Load via `Bitmap.open("/WIDGETS/common/icons/battery-full.png")`

### Typography System

Use `utils.text()` and `utils.S` constants for consistent styling:

```lua
utils.text(x, y, "Label", utils.S.left, utils.S.sml)  -- Small left-aligned
utils.text(x, y, "Value", utils.S.right, utils.S.mid, utils.S.bold)  -- Medium right-aligned bold
utils.textLR(cx, y, "Lat:", "37.7749", 10)  -- Left/right pair around center
```

Constants: `utils.S.base`, `utils.S.left`, `utils.S.right`, `utils.S.sml`, `utils.S.mid`, `utils.S.bold`

### Telemetry Access

Use `getValue(key)` to read sensors. **Always guard against nil values:**

```lua
local tpwr = tonumber(getValue("TPWR")) or 0
local rxBt = tonumber(getValue("RxBt")) or 0
```

**Canonical Telemetry Sensors (Project Truth):**

| Purpose | Sensor Key | Type |
|---------|-----------|------|
| RF link | `TPWR` | Transmitter power (%) |
| Link quality | `RQly` | Link quality (%) |
| RX battery | `RxBt` | Battery voltage (V) |
| RSSI | `1RSS`, `2RSS` | Signal strength (dBm) |
| Antenna | `ANT` | Active antenna |
| RF mode | `RFMD` | RF mode string |
| Flight mode | `FM` | Flight mode |
| Current | `Curr` | Current draw (A) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbarrios83/edgetx-widgets](https://github.com/dbarrios83/edgetx-widgets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
