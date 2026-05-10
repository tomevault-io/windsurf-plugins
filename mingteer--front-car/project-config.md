---
trigger: always_on
description: This project is an embedded application based on the Infineon TC26B microcontroller, presumably related to rear vehicle functions (e.g., reverse assist, rear-view camera processing), named "Back_Car_V1.4".
---

# Project Guide: TC26B Back Car Codebase

## 1. Project Overview
This project is an embedded application based on the Infineon TC26B microcontroller, presumably related to rear vehicle functions (e.g., reverse assist, rear-view camera processing), named "Back_Car_V1.4".

## 2. Key Directory Structure
- `code/`: Stores C source files and header files for core application logic modules. For example, `[code/kalman_filter.c](mdc:code/kalman_filter.c)` implements a Kalman filter.
- `libraries/`: Contains various library files.
    - `infineon_libraries/`: Includes official Infineon low-level libraries (iLLD - Infineon Low Level Drivers) and service layer code for the TC26B chip.
    - `zf_common/`: Contains common project utilities, macro definitions, type definitions, and the crucial central header file `[libraries/zf_common/zf_common_headfile.h](mdc:libraries/zf_common/zf_common_headfile.h)`.
    - `zf_components/`: May contain reusable software components or middleware.
    - `zf_device/`: Stores drivers for specific hardware peripherals (e.g., sensors, displays).
    - `zf_driver/`: Contains further encapsulation or abstraction of underlying hardware drivers.
- `user/`: Stores user-layer code, typically top-level application logic, main program entry points, and user-defined modules referenced in `[libraries/zf_common/zf_common_headfile.h](mdc:libraries/zf_common/zf_common_headfile.h)`.
- `Debug/`: Contains output files generated during compilation and build processes (e.g., `.elf`, `.hex`, map files).
- `.settings/`: Usually contains IDE-specific project configurations (e.g., for Eclipse).

## 3. Core Header File: `zf_common_headfile.h`
`[libraries/zf_common/zf_common_headfile.h](mdc:libraries/zf_common/zf_common_headfile.h)` is the "Swiss Army knife" of this project. It is a central aggregate header file that includes almost all other necessary standard libraries, SDK libraries, custom libraries, and user module header files.

**Importance and Role**:
- **Unified Dependency Management**: Most `.c` files include this file (or a custom header that already includes it) to get almost all required declarations and definitions.
- **Simplified Includes**: Avoids manually including a large number of individual header files in each module.
- **Project Overview**: The include structure of this file also reflects the project's layering and module division.

**Structure**:
The file internally groups included header files clearly using specific comments, for example:
```c
//===================================================C Standard Libraries===================================================
// ... stdio.h, math.h etc.
//===================================================Chip SDK Low Level===================================================
// ... IfxCpu.h, IfxPort.h etc.
//====================================================Open Source Library Common====================================================
// ... zf_common_typedef.h etc.
//===================================================Chip Driver Encapsulation===================================================
// ... zf_driver_adc.h, zf_driver_pwm.h etc.
//===================================================Peripheral Device Drivers===================================================
// ... zf_device_camera.h, zf_device_oled.h etc.
//====================================================User Layer======================================================
// ... image_deal.h, PID.h, All_Init.h etc. // <--- New user module header files are typically added here
```

## 4. Adding New Modules/Features

1.  **File Creation Location**:
    *   New `.c` source files are usually placed in the `code/` or `user/` directory.
    *   Corresponding `.h` header files are also placed in the same directory or a respective `include` subdirectory (if the project uses such a structure). Based on the current structure, `.h` files are in the same directory as `.c` files.

2.  **Writing Header Files (`.h`)**:
    *   **Include Guards**: Standard include guard macros must be used to prevent multiple inclusions of the header file.
        ```c
        #ifndef _MY_NEW_MODULE_H_ // Follow the project's existing naming style, e.g., _FILENAME_H_
        #define _MY_NEW_MODULE_H_

        // (Recommended) Include the central header file if this module requires broad project-level definitions
        // #include "zf_common_headfile.h" // Note the path; if this new header will itself be included by zf_common_headfile.h, this can be omitted or handled carefully to avoid circular include risks.

        // If not directly including zf_common_headfile.h, then necessary other headers are needed
        // e.g., if the new module is in code/ and zf_common_headfile.h is in libraries/zf_common/
        // it might require `#include "../libraries/zf_common/zf_common_headfile.h"` or rely on compiler include path settings.

        // Module-related function declarations, struct definitions, macro definitions, etc.

        #endif // _MY_NEW_MODULE_H_
        ```
    *   **Referencing `zf_common_headfile.h`**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MingTeer/Front_Car](https://github.com/MingTeer/Front_Car) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
