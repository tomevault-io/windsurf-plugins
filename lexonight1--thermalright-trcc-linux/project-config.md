---
trigger: always_on
description: The repo currently carries two parallel source trees on the `dev` branch:
---

# TRCC Linux — Claude Code Project Instructions

## Two Source Trees (read this first)

The repo currently carries two parallel source trees on the `dev` branch:

- **`src/trcc/`** — the **shipping / legacy code**.  Everything users install today runs this.  Full feature set (sensors, setup wizard, autostart, theme download, LED segment displays, `.zt` animations, 4-OS support, etc.).  Every architecture principle below this header describes this tree — SCSI uses `/dev/sgN` + SG_IO on Linux, `DeviceProtocolFactory` exists, `ControllerBuilder` wires things, etc.
- **`src/trcc/next/`** — a **clean-slate rebuild** (12 commits, `c309a55`→`39b3169`).  Proof that a simpler 5-role hexagonal design (Platform / UsbTransport / Device / App / UIs) works end-to-end with one Command bus.  Architecture is complete (3 UIs, two-layer scene cache, video + mask + rotation, tickers wired).  Feature parity with legacy: NO.  Hardware-verified on real devices: NO.  See `memory/project_next_clean_slate.md` for the full status table.

**When the user asks about "the app" or bug fixes for shipping users, work in `src/trcc/`.**  Only touch `src/trcc/next/` when the task is explicitly about the clean-slate rebuild.  Never mix imports between the two trees.

Legacy architecture follows below.

## Architecture — Hexagonal (Ports & Adapters)

### Layer Map
- **Models** (`core/models.py`): Pure dataclasses, enums, domain constants — zero logic, zero I/O, zero framework deps
- **Services** (`services/`): Core hexagon — all business logic, pure Python. `ImageService` delegates to active `Renderer`. `OverlayService` uses injected Renderer for compositing/text.
- **Paths** (`core/paths.py`): Fallback path constants (`DATA_DIR`, `USER_DATA_DIR`). Primary resolution via `PlatformSetup` adapter injected into `Settings`. Zero project imports.
- **Devices** (`core/lcd_device.py`, `core/led_device.py`): Application-layer facades. Strict DI — `RuntimeError` if deps not injected. Zero adapter imports. Delegate to services, return result dicts.
- **Builder** (`core/builder.py`): `ControllerBuilder` — fluent builder, assembles devices with DI. Composition root: imports adapters to inject into services.
- **Views** (`gui/`): PySide6 GUI adapter. `TRCCApp` (thin shell) + `LCDHandler`/`LEDHandler` (one per device).
- **CLI** (`cli/`): Typer CLI adapter (package: `__init__.py` + 8 submodules). Thin wrappers over `LCDDevice`/`LEDDevice`.
- **API** (`api/`): FastAPI REST adapter (package: `__init__.py` + 7 submodules). 49 endpoints. WebSocket preview stream + cloud themes + export. Uses `LCDDevice`/`LEDDevice` from core/.
- **Config** (`conf.py`): `Settings` singleton. `init_settings(platform)` called by composition roots. Single source of truth for mutable app state.
- **Entry**: `cli/` → `trcc_app.py` (TrccApp) → builder.build_device()
- **Protocols**: All protocols implement `send_data()` — SCSI (LCD frames), HID (handshake/resolution), LED (RGB effects + segment displays)
- **Platform** (`core/ports.py`): `OSConfig` dataclass + `Platform` class. OS is data (config), not architecture (class hierarchy). One `Platform` object DI'd everywhere via `builder.os`.
- **OS files** (`adapters/system/{os}_platform.py`): Each exports an `OSConfig` instance (`LINUX_OS`, `WINDOWS_OS`, etc.) + OS-specific functions. ~200 lines each, not 1300.
- **Sensors** (`adapters/system/_base.py`): One `SensorEnumerator` with plugin discovery — tries hwmon, LHM, SMC, sysctl, psutil, pynvml. Each plugin self-guards at runtime.
- **CI**: `release.yml` (Linux RPM/DEB/Arch), `windows.yml` (PyInstaller + Inno Setup), `macos.yml` (PyInstaller + create-dmg)
- **On-demand download**: Theme/Web/Mask archives fetched from GitHub at runtime via `data_repository.py`

### Design Patterns (Used in This Project)
- **Singleton**: `conf.settings` — app-wide state. Widgets read from singleton, never store copies.
- **Factory Method**: `abstract_factory.py` builds protocol-specific device adapters
- **Adapter**: Hexagonal adapters/ — CLI, GUI, API all adapt to the same core services
- **Command**: User actions (button click, terminal command) — log, undo, queue across interfaces
- **Observer**: PySide6 signals broadcast updates from core to views without coupling
- **Strategy**: Swap display/export behaviors without modifying core service logic
- **Template Method**: Concrete method on ABC calls `@abstractmethod` on subclass (e.g. `handshake()` → `_do_handshake()`)
- **Dependency Injection**: Inject dependencies at runtime, never hardcode
- **Repository Pattern**: `data_repository.py` — service layer doesn't know if data comes from file, DB, or API
- **Ports & Adapters**: ABCs as contracts; CLI, GUI, API interact with core the same way
- **DTOs**: `dataclass` for passing data across hexagonal boundaries

### Abstract Base Classes (ABCs)
Two layers: **transport** (raw device I/O) and **adapter** (MVC integration).

#### Transport Layer (`adapters/device/template_method_device.py` + `template_method_hid.py`)
```
UsbDevice (ABC) — handshake() + close()
├── FrameDevice (ABC) — + send_frame()
│   ├── ScsiDevice (adapter_scsi.py)
│   ├── BulkDevice (_template_method_bulk.py)
│   └── HidDevice (ABC, template_method_hid.py) — + build_init_packet, validate_response, parse_device_info

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lexonight1/thermalright-trcc-linux](https://github.com/Lexonight1/thermalright-trcc-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
