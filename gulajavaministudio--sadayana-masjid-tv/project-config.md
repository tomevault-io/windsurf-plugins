---
trigger: always_on
description: Selamat datang di repositori project **Miqotul Khoir TV (MKT)** — aplikasi jam masjid digital dan jadwal sholat berbasis Android TV untuk masjid.
---

# AGENTS.md - Miqotul Khoir TV Contributor Guide

Selamat datang di repositori project **Miqotul Khoir TV (MKT)** — aplikasi jam masjid digital dan jadwal sholat berbasis Android TV untuk masjid.

File ini berisi panduan utama untuk kontributor baru dan AI assistant yang bekerja dengan project Flutter/Dart untuk platform Android TV.

**Last Updated**: April 26, 2026

<!-- markdownlint-disable -->

## Repository Overview

- **Main Application**: `lib/main.dart` — Entry point aplikasi
- **Core Layer**: `lib/core/` — Utilities, constants, dan service layer
- **Data Layer**: `lib/data/` — SQLite models, repositories, dan data sources
- **Domain Layer**: `lib/domain/` — Business logic, entities, dan use cases
- **Presentation Layer**: `lib/presentation/` — UI components, pages, dan state management (Cubit)
- **Tests**: `test/` dengan unit, widget, dan integration test directories
- **Assets**: `assets/` untuk images, fonts, dan resources lainnya
- **Specification**: `spec/` untuk technical specification documents (output @SpecificationArchitect)
- **Planning**: `plan/` untuk feature implementation plans dengan task tracking
- **Documentation**: `docs/` untuk technical specifications dan design documents
- **Database**: SQLite lokal untuk settings, city presets, dan konfigurasi

### State Management Structure

```
lib/presentation/cubits/
├── prayer_times/              # Cubit untuk kalkulasi waktu sholat
├── display_state/            # Cubit untuk state machine display
├── settings/                 # Cubit untuk pengaturan masjid
└── setup_wizard/             # Cubit untuk initial setup flow
```

**Pattern**: Cubit digunakan untuk semua state management, dengan fokus pada local data dan state transitions.

## Completed Features (Production Ready)

| Feature                                              | Completion Date | Tests                          | Status           |
| ---------------------------------------------------- | --------------- | ------------------------------ | ---------------- |
| **Database Infrastructure** (Plan 01)                | 2026-02-18      | 6 unit tests ✅                 | Production Ready |
| **Data Layer** (Plan 02)                             | 2026-02-18      | 16 unit tests ✅ (total: 22)    | Production Ready |
| **Theme System** (Plan 03)                           | 2026-02-18      | 42 unit tests ✅ (total: 64)    | Production Ready |
| **Prayer Time Logic** (Plan 05)                      | 2026-02-19      | Unit tests ✅                   | Production Ready |
| **Prayer Time State** (Plan 06)                      | 2026-02-19      | Cubit tests ✅                  | Production Ready |
| **Display State Logic** (Plan 07)                    | 2026-02-19      | Unit tests ✅                   | Production Ready |
| **Display State Machine** (Plan 08)                  | 2026-02-19      | Cubit tests ✅                  | Production Ready |
| **Setup Wizard Logic** (Plan 09)                     | 2026-02-20      | Cubit tests ✅                  | Production Ready |
| **Setup Wizard UI** (Plan 10)                        | 2026-02-20      | Widget tests ✅                 | Production Ready |
| **Settings Logic** (Plan 11)                         | 2026-02-20      | Unit tests ✅                   | Production Ready |
| **Settings UI** (Plan 12)                            | 2026-02-20      | Widget tests ✅                 | Production Ready |
| **Main Display UI** (Plan 13)                        | 2026-02-20      | Widget tests ✅                 | Production Ready |
| **Kata Mutiara Islam** (Wisdom Quote)                | 2026-03-10      | 14 phases, 257 total tests ✅   | Production Ready |
| **Mode Hemat Daya Tengah Malam** (Midnight Mode)     | 2026-03-16      | 7 phases, 306 total tests ✅    | Production Ready |
| **Alarm Tanda Waktu** (Pre-Adzan & Pre-Iqomah Alert) | 2026-03-17      | 6 phases, 20 new alarm tests ✅ | Production Ready |

### Plan 01 — Database Infrastructure (COMPLETED)

File yang dibuat/dimodifikasi:

| File                                              | Keterangan                                                              |
| ------------------------------------------------- | ----------------------------------------------------------------------- |
| `lib/data/datasources/database_helper.dart`       | Singleton `DatabaseHelper` — schema DDL, seed, migration, testing hooks |
| `assets/data/cities.json`                         | 514 kota/kabupaten, 34 provinsi Indonesia (71.7 KB)                     |
| `tools/generate_cities.py`                        | Script generator dataset kota dari BPS/wilayah-indonesia                |
| `test/data/datasources/database_helper_test.dart` | 6 unit tests dengan `sqflite_common_ffi` in-memory                      |

Dependencies yang ditambahkan:

```yaml
dependencies:
  sqflite: ^2.4.1
  path: ^1.9.1

dev_dependencies:
  sqflite_common_ffi: ^2.4.0+2
```

### Plan 02 — Data Layer (COMPLETED)

File yang dibuat:

| File                                                        | Keterangan                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GulajavaMinistudio/sadayana-masjid-tv](https://github.com/GulajavaMinistudio/sadayana-masjid-tv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
