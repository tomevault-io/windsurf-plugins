---
trigger: always_on
description: provides a shared `QApplication` for tests that touch Qt widgets.
---

# CLAUDE.md — NetBoost Developer Guide

This file provides guidance to Claude Code when working with the NetBoost codebase.

## Project Overview

NetBoost is a Windows gaming network optimizer built with Python 3.13 + PyQt5. It reduces ping spikes, improves FPS consistency, and automates system tweaks for competitive gaming. Requires **administrator privileges** at runtime.

**Target hardware:** Intel i7-13650HX · NVIDIA RTX 4060 Laptop · Intel Wi-Fi 6E AX211 · Windows 11

---

## Version 2 / 2.1 Notes

- **UI theme:** Neo-Dark Glassmorphism QSS — `#00E5FF` cyans, `#05050A` backgrounds, 12px border radii.
- **Animations:** Use `QGraphicsOpacityEffect` + `QPropertyAnimation` for Tabs/Toasts. Never use instant transitions for new UI components.
- **Bug fixes:** false 100% ICMP loss fixed via rapid `ping.exe` fallback when raw socket blocks (`core/ping_monitor.py`); `cp1252` charset bug patched. Maintain coverage threshold.

## Version 2.1 Adaptive Advisor

Adaptive Mode is now **Adaptive Advisor**. It detects loss, ping spikes, and background contention, then queues recommendations in the Monitor tab instead of mutating Windows settings automatically. Any accepted recommendation must route through the existing `MainWindow._apply_wifi`, `_apply_optimizer`, or `_apply_fps` methods so `StateGuard`, transactions, applied badges, diagnostics, and restore paths stay consistent.

---

## Architecture

```
main.py                     # Entry: admin check → crash heal → Qt bootstrap → threads
core/                       # All business logic, no Qt widgets
ui/                         # All PyQt5 widgets, no direct registry/subprocess calls
config/                     # Static JSON (game list)
resources/styles/           # QSS dark theme
tests/                      # pytest unit tests + integration check script
```

**Threading model:**
- Main thread: Qt event loop + all UI updates
- `QThread`: `PingMonitor` → emits `reading(host, ms, timed_out)`
- `QThread`: `ProcessWatcher` → emits `game_launched(str)` / `game_exited(str)`
- `QThreadPool`: slow operations — Wi-Fi latency test (`_LatencyWorker`), RAM poll (`_RamPollWorker`), RAM optimize (`_RamOptimizeWorker`), service stop/start, route trace (`_TraceRouteWorker`), server discovery (`_DiscoverWorker`)

**Signal flow:** Core threads → Qt signals → `MainWindow` slots → UI tab methods. Never call UI methods directly from background threads.

---

## Core Modules

| Module | Responsibility | Key class |
|--------|---------------|-----------|
| `core/admin.py` | UAC elevation via `ShellExecuteW` | `is_admin()`, `elevate()` |
| `core/state_guard.py` | Crash-safe atomic state (`.tmp` → `os.replace`) | `StateGuard` |
| `core/profile_manager.py` | JSON profiles in `%APPDATA%\NetBoost\profiles\` | `ProfileManager` |
| `core/ping_monitor.py` | Raw ICMP socket, fallback to `ping.exe` | `PingMonitor(QThread)` |
| `core/process_watcher.py` | psutil polling every 1.5s; `set_poll_interval(ms)` for runtime adjustment | `ProcessWatcher(QThread)` |
| `core/wifi_optimizer.py` | Intel AX211 registry tweaks | `WifiOptimizer` |
| `core/network_optimizer.py` | TCP registry per interface GUID | `NetworkOptimizer` |
| `core/dns_switcher.py` | netsh DNS switch/restore | `DnsSwitcher` |
| `core/fps_booster.py` | Power plan, P-core affinity, timer res | `FpsBooster` |
| `core/nvidia_optimizer.py` | RTX 4060 registry + nvidia-smi | `NvidiaOptimizer` |
| `core/background_killer.py` | Suspend services/processes on game launch | `BackgroundKiller` |
| `core/bandwidth_manager.py` | DSCP QoS registry + SetPriorityClass | `BandwidthManager` |
| `core/ram_optimizer.py` | EmptyWorkingSet + file cache flush | `RamOptimizer` |
| `core/route_analyzer.py` | tracert parser, bottleneck detection, game server discovery | `_TraceRouteWorker`, `_DiscoverWorker`, `mark_bottlenecks()` |
| `core/network_diagnostics.py` | Honest leg-by-leg ping diagnostic (router / edge / game server) + plain-English verdict; pure logic, no Qt | `ping_target()`, `get_default_gateway()`, `find_active_server_ip()`, `build_verdict()` |
| `core/system_tweaks.py` | Force MSI Mode (GPU + Wi-Fi NIC), disable NDU service, disable NetworkThrottlingIndex | `apply_msi_mode_all`, `disable_ndu_service`, `disable_network_throttling` (+ matching `restore_*`) |
| `core/settings_risk.py` | Risk metadata for every toggle key (pure Python, no Qt) | `get_risk()`, `filter_by_level()` |
| `core/adaptive_engine.py` | Converts telemetry windows into advisor recommendations; no system mutation | `AdaptiveEngine`, `AdaptiveRecommendation` |
| `core/adaptive_advisor.py` | Session-local recommendation queue and settings merge helpers | `RecommendationQueue`, `merge_settings_patch()` |

---

## Critical Design Rules

### StateGuard — always record before destructive operations
Before every registry write, service stop, or process suspend:
```python
state_guard.record_dns_backup(backup)
state_guard.add_paused_service("wuauserv")
state_guard.add_suspended_pid(pid)
```
`restore_all()` is called automatically on clean exit AND on crash recovery (checks if previous PID is dead via `psutil.pid_exists()`).

### Crash recovery flow
1. `StateGuard.check_and_heal()` runs on every startup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gauravsoodtech/net-boost](https://github.com/gauravsoodtech/net-boost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
