---
trigger: always_on
description: OpenProcMon is an open-source Process Monitor implementation for real-time monitoring of process, file and registry activity on Windows. Layered architecture: a kernel miniFilter driver captures events -> the SDK layer talks to the driver over a Filter Manager port -> the GUI layer presents the event data.
---

# OpenProcMon - CLAUDE.md

## Project Overview

OpenProcMon is an open-source Process Monitor implementation for real-time monitoring of process, file and registry activity on Windows. Layered architecture: a kernel miniFilter driver captures events -> the SDK layer talks to the driver over a Filter Manager port -> the GUI layer presents the event data.

The SDK and GUI are a completed ground-up Rust rewrite of the original C++ implementation, which is kept under `cpp-backup/` for reference. The kernel driver is unchanged.

## Architecture

```
┌─────────────────────────────────────┐
│  GUI Layer                          │
│  ├── Rust GPUI GUI (crates/gui/)    │
│  └── C++ WTL GUI (cpp-backup/gui/)  │  [reference]
├─────────────────────────────────────┤
│  SDK Layer                          │
│  ├── Rust SDK (crates/sdk/)         │
│  └── C++ SDK (cpp-backup/sdk)       │  [reference]
├─────────────────────────────────────┤
│  Kernel Driver (kernel/)            │
│  └── miniFilter driver (complete)   │
└─────────────────────────────────────┘
```

## Directory Structure

```
openprocmon/
├── Cargo.toml              # Rust workspace root (members = ["crates/*"])
├── Cargo.lock
├── bin/                    # Prebuilt binaries (stock Process Monitor driver PROCMON24.SYS)
├── kernel/                 # Kernel driver (miniFilter, complete)
│   ├── logsdk.h           # ★ Core: kernel/user-mode interface (structs the Rust SDK mirrors)
│   ├── procmon.c          # Driver entry
│   ├── process.c/h        # Process monitoring callbacks
│   ├── file.c/h           # File-operation monitoring
│   └── reg.c/h            # Registry-operation monitoring
├── crates/                 # ★ Rust workspace
│   ├── sdk/               #   procmon-sdk: driver comms + event parsing + PML read/write
│   │   ├── benches/       #     baseline.rs: CPU+memory bench (see BASELINE.md)
│   │   └── src/           #     monitor/pipeline/parse/event/filter/pml/...
│   ├── gui/               #   procmon-gui: gpui-component GUI on top of the SDK
│   │   ├── locales/       #     en/zh strings (rust-i18n)
│   │   ├── themes/        #     procmon.json (light/dark ThemeConfig + palette)
│   │   └── src/           #     app/model/components/dialogs
│   └── example/           #   procmon-example: console SDK demo (capture/save/replay)
├── docs/
│   └── design/             # GUI design mockups (React/Figma prototypes, reference only)
│       └── gui-design-v1/
└── cpp-backup/             # Original C++ implementation (kept for reference)
    ├── CMakeLists.txt
    ├── cmake/             # CMake build modules
    ├── procmon.sln
    ├── gui/               # C++ WTL GUI (reference)
    │   ├── MainFrm.h      #   Main window
    │   ├── View.h         #   Event list view
    │   ├── dataview.cpp/h #   Data-view management
    │   ├── filterdlg.cpp/h#   Filter dialog
    │   ├── filtermgr.cpp/h#   Filter management
    │   └── propdlg.cpp/h  #   Properties dialog
    └── sdk/procmonsdk/    # C++ SDK (reference)
        ├── sdk.hpp        #   SDK entry point
        ├── kernelsdk.hpp  #   Kernel struct definitions (references logsdk.h)
        ├── monctl.cxx/hpp #   Monitor controller: driver connect, start/stop
        ├── eventmgr.cxx/hpp #  Event manager: queueing and dispatch
        ├── event.cxx/hpp  #   Event object
        ├── eventview.cxx/hpp # Event view interface
        ├── eventfactory.cxx/hpp # Event parsing factory
        ├── drvload.cxx/hpp#   Driver load/unload
        ├── procmgr.cxx/hpp#   Process management
        ├── process.cxx/hpp#   Process info
        ├── fileopt.cxx/hpp#   File-operation parsing
        ├── regopt.cxx/hpp #   Registry-operation parsing
        ├── procopt.cxx/hpp#   Process-operation parsing
        ├── buffer.cxx/hpp #   Thread-safe message buffer
        ├── thread.cxx/hpp #   Thread wrapper
        ├── strmaps.cxx/hpp#   String maps (enum value -> display name)
        ├── utils.cxx/hpp  #   Utility functions
        └── logger.cxx/hpp #   Logging framework
```

## Key Kernel Interface (logsdk.h)

The Rust SDK mirrors these kernel structures exactly (`#pragma pack(1)`):

### Communication constants
- Port: `\\ProcessMonitor24Port`
- Control codes: `CTLCODE_MONITOR=0`, `CTLCODE_THREADPOFILING=1`
- Monitor flags: `CTL_MONITOR_PROC_ON=0x01`, `CTL_MONITOR_FILE_ON=0x02`, `CTL_MONITOR_REG_ON=0x04`

### Core structures
- `LOG_ENTRY` (0x34 bytes, packed) — common header of every event: ProcessSeq, ThreadId, MonitorType, NotifyType, Time, Status, DataLength, …
- `PROCMON_MESSAGE_HEADER` — Filter Manager message header (pack(4))
- `FLTMSG_CONTROL_FLAGS` / `FLTMSG_CONTROL_THREADPROFILING` — control messages

### Event type enums
- `LOG_MONITOR_TYPE`: Process(1), Reg(2), File(3), Profiling(4)
- `LOG_PROCESS_NOTIFY_TYPE`: Init(0), Create(1), Exit(2), ThreadCreate(3), ThreadExit(4), ImageLoad(5), Start(7), Performance(8)
- `LOG_REG_NOTIFY_TYPE`: OpenKeyEx(0) ~ QueryKeySecurity(16)
- File operations: IRP MajorFunction + 20

### Event data structures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [progmboy/openprocmon](https://github.com/progmboy/openprocmon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
