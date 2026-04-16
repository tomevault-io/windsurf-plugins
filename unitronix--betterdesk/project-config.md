---
trigger: always_on
description: > Ten plik jest automatycznie dołączany do kontekstu rozmów z GitHub Copilot.
---

# BetterDesk Console - Instrukcje dla Copilota

> Ten plik jest automatycznie dołączany do kontekstu rozmów z GitHub Copilot.
> Zawiera aktualne informacje o stanie projektu i wytyczne do dalszej pracy.

---

## 📊 Stan Projektu (aktualizacja: 2026-04-10)

### Wersja Skryptów ALL-IN-ONE (v2.4.0)

| Plik | Wersja | Platforma | Status |
|------|--------|-----------|--------|
| `betterdesk.sh` | v2.4.0 | Linux | ✅ ALL-IN-ONE + Node.js only + SSL config + PostgreSQL + Auto mode |
| `betterdesk.ps1` | v2.4.0 | Windows | ✅ ALL-IN-ONE + Node.js only + SSL config + PostgreSQL + Auto mode |
| `betterdesk-docker.sh` | v2.4.0 | Docker | ✅ Interaktywny ALL-IN-ONE + PostgreSQL + Migration |

### Konsole Webowe

| Typ | Folder | Status | Opis |
|-----|--------|--------|------|
| **Node.js** | `web-nodejs/` | ✅ Aktywna (jedyna) | Express.js, EJS, better-sqlite3, CSRF, TOTP 2FA |
| **Flask** | `archive/web-flask/` | 📦 Archived | Python, Jinja2 - przeniesiony do archiwum |

### Serwer BetterDesk (Go)

| Komponent | Folder | Status | Opis |
|-----------|--------|--------|------|
| **Go Server** | `betterdesk-server/` | ✅ Production-ready | Single binary replacing hbbs+hbbr, ~20K LOC Go |
| **Rust (archived)** | `archive/hbbs-patch-v2/` | 📦 Archived | Patched Rust binaries v2.1.3 - przeniesione do archiwum |

### BetterDesk MGMT Client (Tauri + SolidJS) — Operator/Admin Console

| Komponent | Folder | Status | Opis |
|-----------|--------|--------|------|
| **MGMT Client** | `betterdesk-mgmt/` | ⚠️ Alpha (v1.0.0) | Tauri v2, SolidJS, operator/admin desktop app |
| **Installer NSIS** | `src-tauri/target/release/bundle/nsis/` | ✅ | `BetterDesk_MGMT_1.0.0_x64-setup.exe` |
| **Installer MSI** | `src-tauri/target/release/bundle/msi/` | ✅ | `BetterDesk_MGMT_1.0.0_x64_en-US.msi` |

### BetterDesk Agent Client (Tauri + SolidJS) — Endpoint Device Agent

| Komponent | Folder | Status | Opis |
|-----------|--------|--------|------|
| **Agent Client** | `betterdesk-agent-client/` | ⚠️ Alpha (v1.0.0) | Tauri v2, SolidJS, lightweight endpoint agent |
| **Installer NSIS** | `src-tauri/target/release/bundle/nsis/` | ✅ | `BetterDesk_Agent_1.0.0_x64-setup.exe` |

### Serwer Go — Binaries (NIE są w repozytorium, kompilowane lokalnie)

| Platforma | Plik | Status |
|-----------|------|--------|
| Linux x86_64 | `betterdesk-server-linux-amd64` | Kompiluj lokalnie: `go build` |
| Linux ARM64 | `betterdesk-server-linux-arm64` | Kompiluj lokalnie: `GOARCH=arm64 go build` |
| Windows x86_64 | `betterdesk-server.exe` | Kompiluj lokalnie: `GOOS=windows go build` |

---

## 🚀 Skrypty ALL-IN-ONE (v2.4.0)

### Nowe funkcje w v2.4.0

- ✅ **PostgreSQL support** - full PostgreSQL database support for Go server and Node.js console
- ✅ **SQLite → PostgreSQL migration** - built-in migration tool (menu option M/P)
- ✅ **Database type selection** - choose SQLite or PostgreSQL during installation
- ✅ **Docker PostgreSQL** - PostgreSQL container with health checks in docker-compose
- ✅ **Connection pooling** - pgxpool with configurable limits via DSN params
- ✅ **LISTEN/NOTIFY** - real-time event push between Go server instances

### Previous versions

#### v2.3.0
- ✅ **Flask removed** - Flask console deprecated, Node.js is now the only option
- ✅ **SSL certificate configuration** - new menu option C for SSL/TLS setup (Let's Encrypt, custom cert, self-signed)
- ✅ **Security audit fixes** - CSRF protection, session fixation prevention, timing-safe auth, WebSocket auth
- ✅ **TOTP 2FA** - Two-factor authentication with TOTP (otplib)
- ✅ **RustDesk Client API** - dedicated WAN-facing port (21121) with 7-layer security
- ✅ **Address book sync** - full AB storage with address_books table
- ✅ **Operator role** - separate admin/operator roles with different permissions
- ✅ **Desktop connect button** - connect to devices from browser (RustDesk URI handler)

#### v2.2.1
- ✅ Node.js .env config fixes, admin password fixes, systemd fixes

#### v2.2.0
- ✅ Node.js/Flask choice (Flask now deprecated)
- ✅ Migration between consoles
- ✅ Automatic Node.js installation

### Nowe funkcje w v2.1.2

- ✅ **Poprawka systemu banowania** - ban dotyczy tylko konkretnego urządzenia, nie wszystkich z tego samego IP
- ✅ **Poprawka migracji w trybie auto** - migracje bazy danych działają bez interakcji
- ✅ **Weryfikacja SHA256** - automatyczna weryfikacja sum kontrolnych binarek
- ✅ **Tryb automatyczny** - instalacja bez interakcji użytkownika (`--auto` / `-Auto`)
- ✅ **Konfigurowalne porty API** - zmienne środowiskowe `API_PORT`
- ✅ **Ulepszone usługi systemd** - lepsze konfiguracje z dokumentacją

### Funkcje wspólne dla wszystkich skryptów

1. 🚀 **New installation** - full installation from scratch (Node.js only)
2. ⬆️ **Update** - update existing installation
3. 🔧 **Repair** - automatic fix for common issues
4. ✅ **Validation** - check installation correctness
5. 💾 **Backup** - create backups
6. 🔐 **Password reset** - reset admin password
7. 🔨 **Build binaries** - compile from source
8. 📊 **Diagnostics** - detailed problem analysis
9. 🗑️ **Uninstall** - full removal
10. 🔒 **SSL config** - configure SSL/TLS certificates (NEW in v2.3.0)
11. 🔄 **Migrate** - migrate from existing RustDesk Docker (Docker script only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UNITRONIX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
