---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**choco** (CHORD Config Orchestrator) manages [kotekan](https://github.com/kotekan/kotekan/) instances running on a cluster of nodes for the CHORD telescope. It replaces the older **coco** system (reference code in `coco/`) with a simpler, more maintainable architecture.

## Architecture

Single-process Flask app with three concerns:
- **Flask** — web UI with live updates (htmx polling of partial templates; no WebSockets), LDAP auth, JSON API, served by gevent's WSGIServer
- **Queue-based sync system** — a serialized submit path (one lock on the Orchestrator) fans changes out to per-node FIFO queues, processed by a worker pool; workers apply file changes then sync to remote (full restart for base config changes, endpoint POSTs for updatable-only changes)
- **Config directory** — YAML files on disk as source of truth for desired state

Kotekan is deployed and managed on nodes by Ansible. choco only handles monitoring and config management via kotekan's own REST API (port 12048): `/status`, `/config`, `/start`, `/kill`, updatable config endpoints.

choco also fronts two hardware services with monitor badges and control pages (see the services bullets below): the **fpga_master** daemon (F-engine; chive:54321, a `wtl.rest` aiohttp server) and the **power_db** PDB controller (power distribution boards; Infineon high-side switches on a Raspberry Pi, Flask on port 5000).

## Build & Run

The `choco.sh` script wraps common commands. **Keep this script up to date** when install steps, dependencies, or run commands change.

```bash
./choco.sh install   # create venv, install deps, copy config template (--overwrite-configs / --keep-configs)
./choco.sh run       # start choco (extra args forwarded, e.g. ./choco.sh run /path/to/config.yaml)
./choco.sh develop   # loopback-only dev instance: no auth, no TLS, dev/ configs (no root)
./choco.sh test      # run tests (extra args forwarded to pytest)
```

Configuration is in `config.yaml` (gitignored; copy from `config.yaml.template`). Sections: `server`, `configs_dir`, `kotekan`, `sync`, `fpga_master`, `pdb`, `eop`, `bffs`, `eigencal`, `ldap`. See template for all options.

Dependencies are deliberately minimal: core = flask, flask-login, ldap3, gevent, requests, pyyaml; the scientific stack (astropy, numpy, h5py, hdf5plugin) lives in the `[jobs]` extra because only the timer jobs import it (`choco.sh` installs `.[jobs]`, dev venvs `.[dev,jobs]`).  The self-signed TLS fallback shells out to the `openssl` CLI instead of depending on `cryptography`.  Production installs are **pinned and hash-locked**: `requirements.lock` (regenerate with `./choco.sh lock`, review the diff, commit) is what `choco.sh install` feeds pip under `--require-hashes` — each pin carries the sha256 of every artifact PyPI serves for that version (fetched from PyPI's JSON API by `cmd_lock` itself, no pip-tools dependency), so a substituted or tampered file is refused even at the right version; choco itself installs `--no-deps` on top, and the dev venv installs the same lock first for parity.  `pip` itself is in the lock (it is the tool doing the hash verification), and every venv is created with `--upgrade-deps` so the OS-seeded pip is replaced immediately.  `./choco.sh audit` checks every pin against the latest PyPI release and the OSV vulnerability database (read-only, stdlib-only, exits 1 on a known advisory — cron-able); the vendored browser assets in `choco/static/` are outside the lock and are audited by hand against OSV's npm ecosystem.  Before adding a dependency, check the feature isn't a few lines of stdlib or an existing dep away.

## Code Structure

```
choco/
├── app.py          # Flask app factory, gevent WSGI server, SSL, entry point
├── web.py          # Flask routes (blueprint): dashboard, node edit, /update/* JSON API, /service/* pages
├── state.py        # Node (identity, config state, change queue, kotekan REST client), Registry
├── sync.py         # Queue-based sync: ChangeItem, Orchestrator (serialized submit + worker pool)
├── services.py     # FpgaMonitor + PdbMonitor (background polls, control wrappers) + job helpers (job_status, job_logs, timer_status, read_state_json)
├── pdbmap.py       # Master dish-input <-> PDB channel table: CSV load/reload, kotekan dish_inputs cross-check
├── auth.py         # LDAP auth (Flask-Login sessions + LdapAuthenticator direct ldap3 bind, in-memory user store)
├── templates/      # Jinja2 templates (dashboard/edit/login, service pages, _*.html htmx partials)
└── static/         # Vendored assets: pico.min.css, htmx.min.js, idiomorph-ext.min.js, Sortable.min.js
jobs/                               # One subdir per job: units, wrapper, code
├── choco.service               # Main systemd service (Type=notify)
├── eop/                        # EOP broadcast job (runs on choco start + daily timer)
│   ├── choco-eop-broadcast.service
│   ├── choco-eop-broadcast.timer   # Daily at 12:00 UTC
│   ├── eop-broadcast.sh            # Wrapper: finds venv, calls eop_update.py
│   ├── eop_update.py               # EOP pipeline: generate table, merge with state, push to choco

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chord-observatory/choco](https://github.com/chord-observatory/choco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
