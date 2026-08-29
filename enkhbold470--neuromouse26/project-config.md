---
trigger: always_on
description: This file is the single, authoritative guide for AI assistants (Claude, Gemini, etc.) and developers working in this repository.
---

# CLAUDE.md — Micromouse26 ESP32-S3 Firmware Guide

This file is the single, authoritative guide for AI assistants (Claude, Gemini, etc.) and developers working in this repository.

> **🏆 3rd Place Overall · All America Micromouse Contest 2026 (AAMC @ UCLA IEEE)**  
> **MCU:** ESP32-S3 (Xtensa LX7 dual-core, 240 MHz) | **Framework:** Arduino on PlatformIO (stock `espressif32`, Arduino 2.x)

---

## 1. Hardware Architecture & Hardware Context

| Component | Specification & Implementation Details |
|---|---|
| **MCU** | ESP32-S3-WROOM (Xtensa LX7 dual-core, 240 MHz). Single translation unit build (`src/main.cpp`). |
| **Framework & API** | Arduino on PlatformIO using **stock `platform = espressif32`** (Arduino 2.x / ESP-IDF 4.x). LEDC API is 2.x style (`ledcSetup` + `ledcAttachPin` + `ledcWrite`), NOT 3.x `ledcAttach`. |
| **Motor Driver** | DRV8833 dual H-bridge — one driver channel per motor. Fast decay `drive(±speed)` (inactive IN held LOW); half-duty slow decay `brake()` (writes both INs to `MOTOR_PWM_MAX / 2`). |
| **Motors** | GA-N20 brushed DC gear motors — **1:30 gear ratio, 500 RPM @ 6V**, powered by 2S LiPo (7.4V nominal). |
| **Encoders** | Magnetic quadrature encoders (7 CPR disk on motor shaft). Decoded via ESP32-S3 **PCNT hardware peripheral (4× decode)** in `MicromouseEncoderPCNT.h`. Both encoders constructed with `inverted=false` (polarity handled in PinConfig / wiring). `CELL_TICKS` (currently 1373 in `Tuning.h` [F]) is hand-measured per ~180 mm cell. Right ticks are scaled by `RIGHT_ENC_SCALE` (currently `1.0028f` in `PinConfig.h`) via the `rTicks()` wrapper. |
| **IR Sensors** | 4-sensor differential array (SFH4545 narrow 950nm emitters + TEFT4300 phototransistors): LF, L, R, RF. **All four face perpendicular to their target wall** (LF/RF straight forward; L/R ~90° sideways) — see `PinConfig.h` geometry notes. Differential ambient-subtracted reads in `readIR()`. |
| **IMU / Gyro** | MPU-6500 (**I2C** `0x68`) — DLPF=3 (41 Hz BW) to reject PWM harmonics. Integrated Z-axis yaw (`updateYaw()`) for spot turns and forward yaw-hold. |
| **Navigation** | 16×16 flood-fill BFS (`MicromouseMaze.h`). Competition default is classical 4-cell centre goal via `GOAL_CENTRE_*` in `Tuning.h` (also supports a single-cell practice goal). NVS-persisted walls (namespace `mm26`, key `walls`). |
| **Display & UI** | 0.96" 128×64 SSD1306 OLED (I2C `0x3C`) + single tactile Linear Blue Switch (`BUTTON_1=GPIO42`) + Buzzer (`GPIO40`). Rotary encoder used for OLED menu scrolling and Fast Speed adjustment. |
| **Battery** | 300 mAh 2S LiPo (7.4V nominal). Resistor divider `BAT_VDIV_MULT=3.751f` → 0–100% linear SOC. |

---

## 2. Core Development Principles

1. **Single Translation Unit Architecture:** `build_src_filter = +<main.cpp>` means `src/main.cpp` is the only `.cpp` compiled into `[env:main]`. All project headers in `include/` are text-included exactly once. File-scope `static` variables in headers are safe under this pattern.
2. **No Blocking Delays in Control Loops:** The RUN executor in `main.cpp::loop()` paces its position-PID via `micros()`. Never use `delay()` inside any motion control path.
3. **Encoders via Hardware PCNT:** `MicromouseEncoderPCNT.h` uses the ESP32-S3 PCNT peripheral for 4× quadrature decoding with zero CPU overhead. All right-encoder distance reads MUST go through the `rTicks()` wrapper (`main.cpp`).
4. **Arduino 2.x LEDC PWM (200 Hz):** `MOTOR_PWM_FREQ_HZ = 200` (10-bit PWM, 0–1023). 200 Hz empirically yields maximum breakaway torque on this DRV8833 + N20 chassis.
5. **No Magic Numbers:** Every pin assignment lives in `include/PinConfig.h`; every tuning parameter lives in `include/Tuning.h`.
6. **Automatic Gyro Calibration:** `autoCalGyroBeforeStart()` (in `include/OLED.h`) runs automatically before every Explore or Fast Run leg.
7. **Flash On Every Code Edit:** Standard workflow: after making any code edit, compile and flash immediately using `pio run -e main -t upload`. Post-upload chime script is `tools/notify_upload.py`.

---

## 3. Project File Layout

```
neuromouse26/
├── include/
│   ├── README.md                   Module inventory & header include map (start here)
│   ├── Tuning.h                    Every tunable constant (Sections [A]–[H]). Master knob: BASE_BREAKAWAY_PWM
│   ├── PinConfig.h                 Pin mappings, PWM/IR limits, IR_CAL defaults, wall thresholds (+ LEGACY test knobs)
│   ├── IMU.h                       MPU-6500 register stack, bias capture, updateYaw() integration
│   ├── IRSensors.h                 4-sensor IR array, ambient-subtracted reads, EMA filters
│   ├── IRCalibration.h             Per-cm front IR LUT & estimateFrontDistMM()
│   ├── MotionScript.h              RunPhase enums, PhaseStep struct, script[] array, pusher helpers
│   ├── Planner.h                   setupMaze(), senseAndStoreWalls(), buildMoveScript() + straight fusion
│   ├── MicromouseMaze.h            16×16 grid + flood-fill BFS + bestDirectionBiased()
│   ├── MicromouseMotor.h           DRV8833 wrapper (LEDC PWM drive/brake/coast)
│   ├── MicromouseEncoderPCNT.h     ESP32-S3 PCNT 4× quadrature decoder
│   ├── OLED.h                      SSD1306 U8G2 instance, menu, diagnostic screens, auto gyro-cal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enkhbold470/neuromouse26](https://github.com/enkhbold470/neuromouse26) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
