---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NetBooster is a Windows multi-network adapter concurrent download acceleration tool. It uses a SOCKS5 proxy server to intelligently distribute connections across multiple network adapters (Ethernet + Wi-Fi + Mobile Hotspot), achieving bandwidth aggregation for multi-threaded downloads.

**Core Technology Stack:**
- Python 3.10+ with PySide6 (Qt6) for GUI
- QFluentWidgets for Windows 11 Fluent Design UI
- asyncio for SOCKS5 proxy server
- psutil for real-time network traffic monitoring
- PowerShell for Windows network adapter configuration

## Development Commands

### Running the Application

```powershell
# Activate virtual environment
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run application (requires admin privileges - UAC prompt will appear)
python main.py
```

### Building Production Executable

```powershell
pip install nuitka zstandard PySide6-Fluent-Widgets
nuitka --standalone --onefile --enable-plugin=pyside6 --windows-console-mode=disable --windows-uac-admin --windows-icon-from-ico=assets/icon.ico --include-package-data=qfluentwidgets --include-data-dir=assets=assets --python-flag=-O --lto=yes main.py
```

## Architecture Overview

### Three-Layer Architecture

NetBooster implements a **dual-layer architecture** combining physical and application layer scheduling:

```
Application (Steam/IDM/qBittorrent)
         ↓ SOCKS5 Client → 127.0.0.1:1080
ProxyWorker (QThread + asyncio SOCKS5 Server)
         ↓ Round-robin connection distribution
Physical Layer Binding (socket.bind + IP_UNICAST_IF)
         ↓
Multiple Network Adapters → Internet (bandwidth aggregation)
```

### Key Modules

**Entry Point:**
- `main.py` - Application lifecycle: admin privilege check → QApplication creation → MainWindow initialization. Uses factory pattern to delay Qt imports until after QApplication exists.

**Core Proxy Engine:**
- `proxy_worker.py` - **ProxyWorker (QThread)**: Runs asyncio SOCKS5 server in separate thread to avoid blocking Qt event loop
  - **RoundRobinBalancer**: Distributes new connections across selected adapters in round-robin fashion
  - **Physical Layer Binding**: Uses `socket.bind((nic_ip, 0))` + `setsockopt(IPPROTO_IP, IP_UNICAST_IF, nic_index)` to force connections through specific adapters
  - **Async DNS Resolution**: Pre-resolves domains using `loop.getaddrinfo()` to avoid Windows single-adapter DNS deadlock
  - **Traffic Monitoring**: Real-time per-adapter download/upload speed and connection count using psutil

**UI Layer:**
- `ui/main_window.py` - **create_main_window()**: Factory function that returns MainWindow instance
  - **NetworkAdapterTableWidget**: Displays adapters with real-time connection counts
  - **ScanWorker (QThread)**: Background thread for adapter scanning
  - All Qt/qfluentwidgets imports are deferred until factory function is called (critical for avoiding "Must construct QApplication before QWidget" errors)

**Network Utilities:**
- `utils/network_utils.py` - PowerShell-based adapter management
  - `scan_network_adapters()` - Scans connected adapters with IPv4 addresses
  - `set_adapter_metric()` - Modifies adapter metric via PowerShell
  - **Critical**: All PowerShell commands use `InterfaceIndex` (numeric) instead of adapter names to avoid Chinese character encoding issues

**Application Configuration:**
- `utils/app_configurator.py` - Auto-configuration for Steam, IDM, qBittorrent proxy settings

### Signal Flow

**Startup:**
1. User clicks "一键加速" (Boost) → `_start_proxy()`
2. Creates ProxyWorker with selected adapters → `proxy_worker.start()`
3. ProxyWorker emits `started_ok` signal → UI updates status

**Runtime:**
1. SOCKS5 client connects → `_handle_client()` in ProxyWorker
2. RoundRobinBalancer assigns adapter → physical layer binding
3. Every second: `_traffic_monitor()` emits `traffic_signal` with per-adapter stats
4. UI receives signal → updates dashboard and connection counts

**Shutdown:**
1. User clicks "停止加速" (Stop) → `_stop_proxy()`
2. Calls `proxy_worker.stop()` → `loop.call_soon_threadsafe(event.set)`
3. Asyncio loop cancels all client tasks → emits `stopped` signal
4. UI resets to idle state

## Critical Implementation Details

### Physical Layer Binding (The Sacred Foundation)

From `proxy_worker.py:236-241`:
```python
upstream_sock.bind((nic['ip'], 0))
upstream_sock.setsockopt(socket.IPPROTO_IP, IP_UNICAST_IF, struct.pack("!I", nic['index']))
```

This dual binding is **non-negotiable**:
- `bind()` to adapter's IPv4 address
- `setsockopt()` with `IP_UNICAST_IF` (31) to force interface index

**Do not modify** this logic without testing on multiple adapters.

### Chinese Character Encoding

Windows PowerShell commands that use Chinese adapter names cause `subprocess` encoding crashes. Solution:

- **Always** use `InterfaceIndex` (numeric) in PowerShell commands
- Network adapter structure: `{'index': int, 'name': str, 'ip': str}`
- `name` is the Chinese alias (e.g., "以太网", "WLAN") - only for display
- `index` is the numeric InterfaceIndex - used for all PowerShell operations

### Qt Threading Model

**Main Thread:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hypostasis-Cat/HypoMux](https://github.com/Hypostasis-Cat/HypoMux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
