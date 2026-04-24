---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the ClamUI codebase.
---

# CLAUDE.md - ClamUI AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with the ClamUI codebase.

## Project Overview

ClamUI is a modern Linux desktop application providing a graphical user interface for ClamAV antivirus. Built with **PyGObject**, **GTK4**, and **libadwaita** for native GNOME integration.

**Key Facts:**

- Python 3.11+ required
- GTK4/Adwaita UI framework
- ClamAV integration via subprocess (clamscan, clamdscan, freshclam)
- Supports both native and Flatpak installations
- VirusTotal integration for enhanced threat analysis
- MIT licensed

## Repository Structure

```
clamui/
├── src/
│   ├── main.py                 # Application entry point
│   ├── app.py                  # Adw.Application class (lifecycle, views, tray)
│   ├── __init__.py
│   ├── cli/
│   │   └── scheduled_scan.py   # CLI for scheduled scans (systemd/cron)
│   ├── core/                   # Business logic modules
│   │   ├── scanner.py          # ClamAV scanning (sync/async)
│   │   ├── scanner_types.py    # ScanStatus enum, ThreatDetail, ScanResult dataclasses
│   │   ├── scanner_base.py     # Shared scanner utilities (process handling, logging)
│   │   ├── daemon_scanner.py   # clamd daemon integration
│   │   ├── threat_classifier.py # Threat severity/category classification
│   │   ├── scheduler.py        # systemd/cron scheduled scans
│   │   ├── quarantine/         # Threat quarantine system
│   │   │   ├── manager.py      # High-level quarantine operations
│   │   │   ├── database.py     # SQLite metadata storage
│   │   │   ├── file_handler.py # Secure file operations
│   │   │   └── connection_pool.py # SQLite connection pooling
│   │   ├── settings_manager.py # JSON settings persistence
│   │   ├── log_manager.py      # Scan history logging
│   │   ├── notification_manager.py
│   │   ├── battery_manager.py  # Battery status for scheduled scans
│   │   ├── updater.py          # freshclam database updates
│   │   ├── clamav_config.py    # ClamAV configuration parsing
│   │   ├── clamav_detection.py # ClamAV installation/daemon detection
│   │   ├── statistics_calculator.py
│   │   ├── virustotal.py       # VirusTotal API v3 integration
│   │   ├── keyring_manager.py  # Secure API key storage (system keyring)
│   │   ├── clipboard.py        # GTK4 clipboard operations
│   │   ├── result_formatters.py # Scan result formatting (text/CSV)
│   │   ├── sanitize.py         # Input sanitization (log injection prevention)
│   │   ├── path_validation.py  # Path validation and symlink safety
│   │   ├── flatpak.py          # Flatpak detection and host command wrapping
│   │   ├── i18n.py             # Internationalization (gettext setup)
│   │   └── utils.py            # General utility functions
│   ├── profiles/               # Scan profile management
│   │   ├── profile_manager.py  # CRUD operations, validation, import/export
│   │   ├── profile_storage.py  # JSON persistence
│   │   └── models.py           # ScanProfile dataclass
│   └── ui/                     # GTK4/Adwaita UI components
│       ├── window.py           # Main application window
│       ├── scan_view.py        # Scanning interface
│       ├── update_view.py      # Database update view
│       ├── logs_view.py        # Scan history (with pagination)
│       ├── quarantine_view.py  # Quarantine management (with pagination)
│       ├── statistics_view.py  # Statistics dashboard
│       ├── components_view.py  # ClamAV components status
│       ├── preferences/        # Modular preferences system
│       │   ├── __init__.py
│       │   ├── base.py         # PreferencesPageMixin with shared utilities
│       │   ├── window.py       # PreferencesWindow orchestrating all pages
│       │   ├── behavior_page.py    # Close behavior, notifications, tray
│       │   ├── database_page.py    # Freshclam settings
│       │   ├── exclusions_page.py  # Exclusion pattern management
│       │   ├── onaccess_page.py    # On-access scanning configuration
│       │   ├── save_page.py        # Save & apply with permission elevation
│       │   ├── scanner_page.py     # Clamd configuration
│       │   ├── scheduled_page.py   # Scheduled scan configuration
│       │   └── virustotal_page.py  # VirusTotal API configuration
│       ├── profile_dialogs.py  # Profile create/edit dialogs
│       ├── scan_results_dialog.py  # Scan results with quarantine actions
│       ├── virustotal_results_dialog.py # VirusTotal detection details
│       ├── virustotal_setup_dialog.py   # VirusTotal API key setup
│       ├── close_behavior_dialog.py     # Window close behavior dialog
│       ├── file_export.py      # Reusable file export dialog helper
│       ├── pagination.py       # PaginatedListController for large lists
│       ├── view_helpers.py     # Shared UI utilities (status, empty states)
│       ├── utils.py            # UI utility functions
│       ├── tray_manager.py     # System tray subprocess manager
│       ├── tray_service.py     # System tray D-Bus service
│       ├── tray_icons.py       # Tray icon resource management
│       ├── tray_indicator.py   # GTK3 tray indicator (legacy)
│       └── fullscreen_dialog.py
├── tests/                      # Test suite (mirrors src structure)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linx-systems/clamui](https://github.com/linx-systems/clamui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
