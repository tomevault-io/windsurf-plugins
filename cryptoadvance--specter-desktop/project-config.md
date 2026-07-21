---
trigger: always_on
description: Guidelines for AI contributors working on specter-desktop.
---

# AGENTS.md — Specter Desktop

Guidelines for AI contributors working on specter-desktop.

## Project Overview

Specter Desktop is a GUI for Bitcoin Core & Electrum optimized for airgapped hardware wallets. It's a Flask web app that manages multisig wallets, coordinates hardware wallet signing via HWI, and handles PSBT-based transaction workflows.

**License:** MIT
**Stack:** Python 3.9-3.10, Flask, Jinja2 templates, plain JavaScript (no frameworks), JSON file persistence, PyInstaller for desktop builds
**Status:** Reviving. Last release v2.1.1 (2025-01-03). Release pipeline recently migrated to GitHub Actions. See "Current State" section at the bottom.

## Architecture (Quick Reference)

```
src/cryptoadvance/specter/
├── server.py          # Flask app factory, create_and_init()
├── cli/               # CLI entry point (specter command)
├── managers/           # Core business logic
│   ├── device_manager.py    # Hardware wallet device registry
│   ├── wallet_manager.py    # Wallet CRUD + balance/tx tracking
│   ├── node_manager.py      # Bitcoin Core / Electrum connections
│   └── user_manager.py      # Auth + RBAC
├── rpc.py             # Bitcoin Core JSON-RPC client
├── persistence.py     # JSON file storage (no SQL)
├── device.py          # Device domain model + HWI glue
├── wallet/            # Wallet domain logic, descriptors, address/tx helpers
├── util/
│   └── psbt.py        # PSBT utilities (parse, analyze, finalize)
├── devices/
│   └── hwi/           # HWI-based hardware wallet drivers (Jade, KeepKey, DIY)
├── hwi_rpc.py         # HWI JSON-RPC wrapper
├── hwi_server.py      # HWI bridge subprocess (for GUI builds)
├── templates/         # Jinja2 HTML templates
├── static/            # CSS/JS/images
└── services/          # Extension system (specterext namespace packages)
```

**Key patterns:**
- Manager pattern: each domain has a `*Manager` class that owns the lifecycle
- JSON file persistence in `~/.specter/` (devices, wallets, nodes as JSON files)
- Extensions via `specterext` namespace packages with their own Flask blueprints
- HWI bridge for hardware wallet communication (Trezor, Ledger, ColdCard, etc.)
- PSBT workflow: construct → sign (hardware) → broadcast

## Running from Source

```bash
# Prerequisites (Ubuntu/Debian)
sudo apt install libusb-1.0-0-dev libudev-dev libffi-dev libssl-dev build-essential

# Clone and set up
git clone https://github.com/cryptoadvance/specter-desktop.git
cd specter-desktop
pip3 install virtualenv
virtualenv --python=python3.10 .env
source .env/bin/activate
pip3 install -r requirements.txt --require-hashes
pip3 install -e .

# Run dev server
python3 -m cryptoadvance.specter server --config DevelopmentConfig --debug
# → http://127.0.0.1:25441/
```

**Python version:** 3.9 or 3.10 required. 3.11+ will break on some dependencies.

## CI/CD

**GitHub Actions only.** Cirrus CI and GitLab CI were retired in 2026-Q2 — see `docs/ci-migration-evidence.md` for the cutover evidence and `docs/continuous-integration.md` for the active topology.

### Workflows

| Workflow | File | Trigger | Purpose |
|----------|------|---------|---------|
| Tests | `test.yml` | PR, push | pytest + Cypress + extension smoketest (3 jobs) |
| Release | `release.yml` | Tag push `v*` | pip + specterd + Electron for Linux/Win/macOS + GPG-sign `SHA256SUMS` |
| Black Linter | `zblack.yml` | PR, push | `psf/black@26.3.0` action pinned to Black 22.3.0, python-3.12 |
| TOC Generator | `toc.yml` | Push | Auto-generates TOCs for `README.md`, `docs/faq.md`, `docs/development.md` |
| Docker Push | `docker-push.yml` | Push to any branch | Multi-arch image → `ghcr.io/cryptoadvance/specter-desktop:<branch>` |
| Docker Tag | `docker-tag.yml` | Tag push `v*` | Multi-arch image → `ghcr.io/cryptoadvance/specter-desktop:<tag>` |
| Extension Compat | `extension-compat.yml` | PR touching `requirements.*`/`pyproject.toml` | Installs full lock, imports every bundled extension, runs `pip check` |
| specterd Build Smoke | `test-specterd-build.yml` | PR touching `pyinstaller/`, `requirements*`, `src/**` | Builds specterd on Linux and runs `--help` |
| Electron Smoke | `electron-smoketest.yml` | PR touching `pyinstaller/electron/**` | Smoke test Electron packaging |

All workflows use public GitHub-hosted runners (`ubuntu-latest` / `ubuntu-22.04` / `windows-latest` / `macos-14`). **No private runners.**

### Test workflow — `test.yml`

Three jobs on `ubuntu-22.04`:
1. **`test`** — pytest with `--cov=cryptoadvance`, 45-min timeout. Installs system deps inline; no custom image. Caches bitcoind/elementsd via `actions/cache@v4` keyed on `runner.os × runner.arch × hash(pyproject.toml, install_noded.sh, bitcoin_SHA256SUMS, elements_SHA256SUMS)` with `save-always: true`.
2. **`cypress`** — `./utils/test-cypress.sh --debug run` inside `ghcr.io/cryptoadvance/specter-desktop/cypress-python-jammy@sha256:<digest>`, 30-min timeout, `--shm-size=2g`.
3. **`extension-smoketest`** — byte-compatible port of the old Cirrus smoketest, 15-min timeout.

`tests/install_noded.sh` GPG-verifies upstream `SHA256SUMS.asc` and checks tarball SHA256 against the committed trust anchors on every run (cold cache AND cache hit).

### Release pipeline — `.github/workflows/release.yml`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cryptoadvance/specter-desktop](https://github.com/cryptoadvance/specter-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
