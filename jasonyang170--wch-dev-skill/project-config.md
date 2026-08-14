---
trigger: always_on
description: > Core rules, mapping table, pitfalls, recipes index, and execution workflow are all in `SKILL.md`.
---

# AGENTS.md — Supplementary Agent Guide

> Core rules, mapping table, pitfalls, recipes index, and execution workflow are all in `SKILL.md`.
> This file covers **only** conventions and tooling guidance not present in `SKILL.md`. Do not duplicate content.

## Project Context

**Language**: C · **Target**: WCH full-series RISC-V / ARM / 8051 SoC · **Toolchain**: MounRiver Studio (GCC RISC-V/ARM) or Keil C51/MDK

## Architecture Quick Reference

| Architecture | Chips | Header | Build Tool | Linker Script |
|---|---|---|---|---|
| RISC-V (BLE) | CH57x, CH58x/CH59x | `CH57x_common.h`, `CH58x_common.h`, `CH59x_common.h` | MounRiver | `Ld/Link.ld` |
| RISC-V (General) | CH32V103, CH32V20x, CH32V307, CH32V407 | `ch32v10x.h`, `ch32v20x.h`, `ch32v30x.h`, `ch32v4x7.h` | MounRiver | `Ld/Link.ld` |
| RISC-V (Low-Cost) | CH32V003, CH32V006, CH32L103 | `ch32v00x.h`, `ch32l103.h` | MounRiver | `Ld/Link.ld` |
| RISC-V (USB-PD) | CH32X035, CH32X315, CH641, CH643 | `ch32x035.h`, `ch32x3x5.h`, `ch643.h`, `ch641.h` | MounRiver | `Ld/Link.ld` |
| RISC-V (High-Perf) | CH32H417 | `ch32h417.h` | MounRiver | `Ld/Link.ld` |
| RISC-V (Ethernet) | CH569 | `CH56x_common.h` | MounRiver | `Ld/Link.ld` |
| ARM Cortex-M3 | CH32F103, CH32F20x, CH32M030 | `ch32f10x.h`, `ch32f20x.h` | Keil MDK / MounRiver | `Ld/Link.ld` or `.sct` |
| ARM7TDMI | CH561, CH563 | `CH561SFR.H`, `CH563SFR.H` | Keil MDK | Scatter file (`.sct`) |
| 8051 | CH543-CH559 | `CH5xx.H` | Keil C51 / SDCC | N/A (bank switching) |

## Code Generation Conventions

### File Naming
- Source files: `*.c`, headers: `*.h`
- BLE application files: `<role>_main.c` (e.g., `peripheral_main.c`, `central_main.c`)
- BLE service profiles: `<service_name>.c` in `Profile/` directory
- BLE configuration: `config.h` in project `APP/include/`
- StdPeriphDriver: `CH*_<peripheral>.c` (e.g., `CH57x_gpio.c`, `CH32V10x_tim.c`)

### Include Patterns

**CH57x BLE projects:**
```c
#include "CONFIG.h"
#include "HAL.h"
#include "CH57xBLE_LIB.h"
#include "peripheral.h"       // or central.h, etc.
#include "devinfoservice.h"
#include "gattprofile.h"
```

**CH57x non-BLE projects:**
```c
#include "CH57x_common.h"     // pulls in all peripheral headers
```

**CH58x/CH59x BLE projects:**
```c
#include "CONFIG.h"
#include "HAL.h"
#include "CH58xBLE_LIB.h"     // or CH59xBLE_LIB.h
#include "peripheral.h"
```

**CH32V general-purpose (StdPeriphDriver):**
```c
#include "ch32v10x.h"         // or ch32v20x.h, ch32v30x.h, ch32v4x7.h
#include "ch32v10x_gpio.h"
#include "ch32v10x_usart.h"
// or use: #include "ch32v10x_conf.h"  // pulls in all peripheral headers
```

**CH32F ARM (StdPeriphDriver):**
```c
#include "ch32f10x.h"         // or ch32f20x.h
#include "ch32f10x_gpio.h"
```

**CH32X/CH64x (StdPeriphDriver):**
```c
#include "ch32x035.h"
#include "ch32x035_gpio.h"
```

**CH561/CH563 ARM7TDMI (register-level):**
```c
#include "CH561SFR.H"         // or CH563SFR.H
#include "SYSFREQ.H"
```

**8051:**
```c
#include <CH554.H>            // or CH543.H, CH549.H, etc.
#include <stdio.h>
```

### Standard BLE Project Structure (CH57x/CH58x/CH59x)
```
MyBLEProject/
├── APP/
│   ├── include/
│   │   ├── config.h          # BLE stack configuration
│   │   └── peripheral.h      # Application header
│   ├── peripheral_main.c     # main() and init
│   └── peripheral.c          # Application logic, callbacks
├── Profile/
│   ├── include/
│   │   ├── devinfoservice.h
│   │   └── gattprofile.h
│   ├── devinfoservice.c      # Device Information Service
│   └── gattprofile.c         # Custom GATT profile
├── HAL/                      # Linked from resources/EXAM/BLE/HAL/
├── Ld/Link.ld                # Linked from resources/EXAM/SRC/Ld/
├── RVMSIS/                   # Linked from resources/EXAM/SRC/RVMSIS/
├── Startup/                  # Linked from resources/EXAM/SRC/Startup/
├── StdPeriphDriver/          # Linked from resources/EXAM/SRC/StdPeriphDriver/
├── .project                  # MounRiver Studio project file
└── *.wvproj                  # MounRiver Studio workspace project
```

### Standard Main Loop Pattern — BLE (CH57x)
```c
int main(void) {
    SetSysClock(CLK_SOURCE_PLL_60MHz);

#if (defined(DCDC_ENABLE)) && (DCDC_ENABLE == TRUE)
    PWR_DCDCCfg(ENABLE);
#endif

#ifdef DEBUG
    UART1_DefInit();
    PRINT("CH57x BLE init...\n");
#endif

    CH57X_BLEInit();
    HAL_Init();
    GAPRole_PeripheralInit();
    Peripheral_Init();

    Main_Circulation();  // never returns
    return 0;            // unreachable
}
```

### Standard Main Loop Pattern — CH32V (StdPeriphDriver)
```c
int main(void) {
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    SystemCoreClockUpdate();
    Delay_Init();
    USART_Printf_Init(115200);
    printf("CH32V system clock: %d Hz\n", SystemCoreClock);

    // Peripheral init here
    GPIO_Config();
    USART_Config();

    while(1) {
        // Application loop
    }
}
```

### Standard Main Loop Pattern — 8051
```c
void main(void) {
    // Safe mode for protected registers
    SAFE_MOD = 0x55;
    SAFE_MOD = 0xAA;
    // Configure clock, GPIO, etc.
    // ...
    SAFE_MOD = 0x00;

    // Enable interrupts
    EA = 1;

    while(1) {
        // Application loop
    }
}
```

### Interrupt Handler Templates

**RISC-V (CH57x/CH58x/CH59x — BLE chips):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasonYANG170/wch-dev-skill](https://github.com/JasonYANG170/wch-dev-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
