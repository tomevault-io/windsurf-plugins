---
trigger: always_on
description: **Fiscalberry** is a multi-platform print server (Windows/Linux/Android) that translates JSON commands into printer-specific protocols via SocketIO/RabbitMQ. It supports fiscal printers (Hasar/Epson) and ESC/POS receipt printers.
---

# Fiscalberry AI Agent Instructions

## Project Overview

**Fiscalberry** is a multi-platform print server (Windows/Linux/Android) that translates JSON commands into printer-specific protocols via SocketIO/RabbitMQ. It supports fiscal printers (Hasar/Epson) and ESC/POS receipt printers.

**Key Architecture Pattern**: Fiscalberry acts as a 3-in-1 system:
1. **Protocol**: Standardized JSON format for all printer commands
2. **Server**: SocketIO for management + RabbitMQ for job queuing
3. **Driver**: Translates JSON → printer-specific commands (Hasar, Epson, ESC/POS)

## Critical Architecture Components

### Entry Points (3 Modes)
- **GUI Mode**: `src/fiscalberry/gui.py` → `FiscalberryApp` (Kivy-based, cross-platform)
- **CLI Mode**: `src/fiscalberry/cli.py` → `ServiceController` (headless servers/Raspberry Pi)
- **Android Service**: `src/fiscalberryservice/android.py` (background service)

### Core Services (Singleton Pattern)
```python
ServiceController  # Orchestrates SocketIO + RabbitMQ lifecycle
├── FiscalberrySio  # SocketIO client for real-time management commands
└── RabbitMQProcessHandler → RabbitMQConsumer  # Job queue processing
    └── ComandosHandler → Printer drivers
```

**CRITICAL**: `ServiceController` is a singleton - always access via existing instance, never instantiate directly in new code.

### Configuration System (`Configberry`)
- **File**: `config.ini` (INI format, environment-aware)
- **Priority Logic**: Local `config.ini` > SocketIO-provided config > Defaults
- **Adoption Flow**: App starts "unadopted" → SocketIO `start_rabbit` event → writes `[Paxaprinter]` section → becomes "adopted"
- **Listener Pattern**: Components register with `configberry.add_listener()` to react to config changes

### Print Job Flow
```
JSON Command (SocketIO/RabbitMQ) 
  → ComandosHandler.runTraductor()
  → Driver-specific translation (Hasar/Epson/ESC/POS)
  → Printer (Serial/USB/Network/Bluetooth)
  → Response JSON back to client
```

## Android-Specific Architecture

### Build System
- **Tool**: Buildozer (Python-for-Android wrapper)
- **Build Command**: `buildozer android debug` (15-20 min compile time)
- **Custom Recipes**: `my_recipes/` for native dependencies (jpeg, kivy patches)
- **APK Output**: `bin/fiscalberry-{version}-{arch}-debug.apk`

### Permission System (Android 12+ Critical)
- **Runtime Permissions**: Bluetooth requires explicit user grant via `ActivityCompat.requestPermissions()`
- **Code Location**: `src/fiscalberry/common/android_permissions.py`
- **Pattern**: Always use dual-method approach (p4a + ActivityCompat fallback)

### Android Service Pattern
```python
# Main app starts service in background
from android import AndroidService  # OR jnius autoclass fallback
service.start(mActivity, '')  # Keeps RabbitMQ alive when app in background
```

## Printer Driver Architecture

### Driver Loading Pattern (ComandosHandler.py)
```python
# Dynamic driver instantiation based on config.ini [IMPRESORA] section
driver = config['driver'].lower()  # "hasar", "epson", "network", "bluetooth"
marca = config['marca']  # Printer brand/model

if driver == "hasar":
    from Traductores.TraductorFiscal import TraductorHasar
elif driver == "network":
    from escpos import printer
    driver_obj = printer.Network(host=config['host'], port=config['port'])
elif driver == "bluetooth":  # NEW: Android-specific
    from fiscalberry.common.bluetooth_printer import BluetoothPrinter
    driver_obj = BluetoothPrinter(mac_address=config['mac_address'])
```

**NEW PATTERN (Bluetooth)**: Uses Android BluetoothSocket via `jnius` → SPP UUID `00001101-...` → implements `escpos` interface.

### Printer Detection (Android)
- **USB**: Via `pyusb` + Android USB OTG permissions
- **Bluetooth**: Scans bonded devices + discovery via `BluetoothAdapter.getDefaultAdapter()`
- **Code**: `src/fiscalberry/common/printer_detector.py`

## Developer Workflows

### Local Development Setup
```bash
# Create virtual environments (2 separate envs for GUI vs CLI)
python3 -m venv venv.cli
source venv.cli/bin/activate
pip install -r requirements.cli.txt

# OR for GUI development
python3 -m venv venv.kivy
source venv.kivy/bin/activate
pip install -r requirements.kivy.txt

# Run modes
python src/fiscalberry/cli.py          # CLI mode
python src/fiscalberry/gui.py          # GUI mode
```

### Android Build & Deploy

**⚙️ Prerequisitos:**
- Python 3.8+ (preferiblemente 3.11 o 3.12)
- Android SDK/NDK (se instala automáticamente por Buildozer)
- Java JDK 11 o superior
- Git, zip, unzip, autoconf, libtool, pkg-config

**📦 Setup inicial (solo primera vez):**
```bash
# 1. Crear entorno virtual dedicado para buildozer
python3 -m venv venv.buildozer
source venv.buildozer/bin/activate

# 2. Instalar buildozer y Cython
pip install --upgrade pip
pip install buildozer cython==0.29.36

# 3. Verificar configuración
buildozer android debug --verbose  # Primera vez descarga SDK/NDK (~2-3 GB)
```

**🔨 Comandos de compilación:**

```bash
# Activar entorno buildozer
source venv.buildozer/bin/activate

# BUILD LIMPIO (cuando cambias requirements.txt o buildozer.spec)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paxapos/fiscalberry](https://github.com/paxapos/fiscalberry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
