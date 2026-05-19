---
trigger: always_on
description: Automates deployment of gen_server.py updates on the Olimex.
---

# PiGenny Generator Controller - Project Context

## Overview
PiGenny is a hybrid Raspberry Pi + Olimex iMX233 system that automatically starts a backup generator when battery state of charge drops below a threshold. This replaces the older single-board Arch Linux ARM system.

## Architecture

### Hardware Components
- **Raspberry Pi** (momspi.local / 10.2.242.1)
  - RS-485 communication with LuxPower/GSL inverter (19200 baud on /dev/ttySC1)
  - TCP client to Olimex generator controller
  - Runs control logic and monitoring (monitor.py)

- **Olimex iMX233-OLinuXino-MAXI** (10.2.242.109)
  - I2C control of MOD-IO relay board (address 0x58)
  - TCP server for generator commands (gen_server.py on port 9999)
  - Direct relay control for generator start/stop/charger

- **LuxPower/GSL Inverter**
  - RS-485 Modbus RTU (slave address 1)
  - Reports battery SOC, voltage, PV power, charge/discharge

- **Generator**
  - Controlled via 4 relays: IGN (ignition), START (starter motor), GLOW (glow plugs), CHARGER (AC transfer)
  - Status feedback via MOD-IO input register 0x20

### Network
- Direct-wire Ethernet between Pi and Olimex
- Pi: 10.2.242.1
- Olimex: 10.2.242.109
- Isolated network (no external connectivity)

## CRITICAL: Understanding Battery Charging

### Inverter Register Interpretation

**IMPORTANT:** The LuxPower inverter has separate registers for different power flows:

- **Register 10 (charge_power_w)**: PV SOLAR charging power ONLY
  - This shows how much power is coming from solar panels to charge the battery
  - This will be 0W at night or when generator is the charging source

- **Register 11 (discharge_power_w)**: Battery discharge to loads
  - Power being drawn from battery to supply house loads

- **Register 9 (pv_power_w)**: Total solar panel output

### How to Tell if Generator Charging is Working

**DO NOT rely on charge_power_w register!** It only tracks solar charging.

When the generator is running and charging the batteries, you should see:
1. **SOC (State of Charge) increasing** - e.g., 21% → 28% over time
2. **Battery voltage increasing** - e.g., 51.5V → 53.4V
3. **Discharge power lower than house load** - some load being supplied by generator

**If you see:** SOC rising, voltage rising, but charge_power_w = 0
- **This is NORMAL** - generator is charging, solar is not
- The inverter likely doesn't report AC/grid charging power in the Modbus registers we're reading

**If you see:** SOC falling, voltage falling, charge_power_w = 0
- **This is a problem** - no charging is happening
- Check relay states, AC wiring, generator running status

## Generator Control Sequence

### Start Sequence (gen_server.py)
1. Phase 1: Reset (0b0000) - 1 second
2. Phase 2: IGN on (0b1000) - 0.5 seconds
3. Phase 3: IGN + START (0b1001) - 10 seconds (main crank)
4. Phase 4: IGN + GLOW + START (0b1101) - 2 seconds (glow-assisted crank)
5. Phase 5: IGN only (0b1000) - 4 seconds (coast and verify)
6. Check status register: if running (status == 3):
   - 60 second warmup at idle
   - 20 second AC stabilization
   - Enable charger relay (0b1010 = IGN + CHARGER)
7. If not running: emergency shutdown (0b0000)

### Stop Sequence (gen_server.py)
1. Disconnect charger (0b1000 = IGN only)
2. Run at idle for 3 minutes cooldown
3. Full shutdown (0b0000)

**Note:** 3-minute cooldown is only needed if generator was under load. If generator was running but not actually charging (e.g., relay on but AC not connected), immediate shutdown is fine.

## Known Issues and Fixes

### Thread Leak Bug (Fixed 2026-01-02)

**Problem:** gen_server.py had a thread leak where each TCP connection created a new thread that was never cleaned up. After ~5 hours of hourly status checks, the system would exhaust thread limits and crash with "can't start new thread" error.

**Symptoms:**
- Pi logs show "Broken pipe" errors when trying to connect to Olimex
- gen_server.log shows "Server error: can't start new thread"
- Emergency relay shutdown triggered
- System in ERROR state, generator won't start even when SOC is low

**Fix Applied:**
- Added thread tracking and cleanup in gen_server.py
- Implemented MAX_CONCURRENT_CONNECTIONS limit (5)
- Active threads are cleaned up after each connection completes
- Added logging for thread cleanup operations

**File:** gen_server.py lines 51-61, 285-293, 353-382

**Deployment Note:** Restarting gen_server.py immediately shuts down the generator (relays set to 0b0000 on startup). Only deploy updates when generator is not needed or after a proper stop sequence.

## Maintenance Tools

### update_genserver.py (Olimex)
Automates deployment of gen_server.py updates on the Olimex.

**Usage:**
```bash
# On Olimex (run as root or with sudo)
sudo python2 /usr/local/bin/update_genserver.py [--source /path/to/gen_server.py]
```

**What it does:**
1. Verifies source file exists
2. Copies to /usr/local/bin/gen_server.py
3. Kills old process (SIGTERM, then SIGKILL if needed)
4. Waits for rc.local to auto-restart
5. Verifies new process started and is responding

**Default source:** `/home/derekja/gen_server.py`

**Remote deployment from Pi:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekja/pigenny](https://github.com/derekja/pigenny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
