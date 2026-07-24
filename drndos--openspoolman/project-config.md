---
trigger: always_on
description: This document is for AI coding agents (and humans) making changes to **OpenSpoolMan**.
---

# agents.md — OpenSpoolMan

This document is for AI coding agents (and humans) making changes to **OpenSpoolMan**.
Follow it as the default “operating manual” when creating PRs.

## 1) Project intent (do not drift)
OpenSpoolMan augments SpoolMan with Bambu Lab / AMS awareness and optional NFC workflows:
- Keep all operations **local-first** (LAN where possible).
- NFC is **optional**; the web UI must remain fully usable without NFC.
- The system is an “adapter + UI” on top of SpoolMan, not a replacement.

If a proposed change alters any of these fundamentals, stop and propose it as a design discussion first.

---

## 2) Non-negotiables (hard rules)
### Security & privacy
- Never commit secrets (printer access codes, API keys, cookies, tokens, personal URLs).
- Do not log secrets. Mask them if you must log configuration.
- Treat everything coming from MQTT / HTTP as untrusted input.

### Backwards compatibility
- Preserve existing env vars and default behaviors unless explicitly versioned.
- UI behavior must remain functional for:
  - No NFC usage
  - SpoolMan available/unavailable (graceful handling)
  - AUTO_SPEND disabled

### Reliability
- Network calls must have **timeouts**, error handling, and retry/backoff where appropriate.
- Never introduce busy loops. Prefer event-driven updates or bounded polling.

---

## 3) Repository map (high-level)
Key folders/files you will interact with:
- `app.py` / `wsgi.py`: application entry points
- `templates/`, `static/`: server-rendered UI assets
- `mqtt_bambulab.py`: Bambu printer connectivity (LAN / MQTT)
- `spoolman_client.py`, `spoolman_service.py`: SpoolMan integration layer
- `filament.py`, `filament_usage_tracker.py`, `print_history.py`: domain logic
- `scripts/`: helper scripts (e.g., initialization / tooling)
- `data/`: runtime artifacts (DBs, mismatch logs)
- `tests/`: Python tests
- `e2e/`, `playwright.config.js`, `package.json`: end-to-end UI tests
- `docker-compose.yaml` / `compose.yaml` / `Dockerfile`: containerization
- `helm/openspoolman`: Helm chart

---

## 4) How to run locally (known-good paths)

### 4.1 Local Python run (development)
1. Configure environment (see §5). Create `config.env` from `config.env.template` or export env vars.
2. Start the server:
   - `python wsgi.py`

Notes:
- Default listen port is `8001` (to avoid clashing with SpoolMan).
- Depending on SSL mode and mapping you may also access `https://<host>:8443`.

### 4.2 Docker (deployment / reproducible dev)
- Configure env vars, then:
  - `docker compose up -d`

Use `docker compose port openspoolman 8001` to see mapped host port if needed.

### 4.3 Kubernetes (Helm)
- Use the bundled chart:
  - `helm dependency update helm/openspoolman`
  - `helm upgrade --install openspoolman helm/openspoolman -f values.yaml --namespace openspoolman --create-namespace`
- Validate:
  - `kubectl get pods -n openspoolman`

---

## 5) Configuration contract (environment variables)
### Required / core
- `OPENSPOOLMAN_BASE_URL`
  - HTTPS URL where OpenSpoolMan is reachable
  - **No trailing slash**
  - Required for NFC writes
- `PRINTER_ID`
  - Printer settings → Setting → Device → Printer SN
- `PRINTER_ACCESS_CODE`
  - Setting → LAN Only Mode → Access Code
  - (LAN Only Mode toggle may stay off)
- `PRINTER_IP`
  - Setting → LAN Only Mode → IP Address
- `SPOOLMAN_BASE_URL`
  - URL of SpoolMan without trailing slash

### Feature toggles
- `AUTO_SPEND`
  - `True` enables legacy slicer-estimate tracking.
- `TRACK_LAYER_USAGE`
  - `True` switches to per-layer tracking/consumption **only if** `AUTO_SPEND=True`.
  - If `AUTO_SPEND=False`, tracking remains disabled regardless of `TRACK_LAYER_USAGE`.
- `DISABLE_MISMATCH_WARNING`
  - `True` hides mismatch warnings in the UI (still detected and logged).
- `CLEAR_ASSIGNMENT_WHEN_EMPTY`
  - `True` clears SpoolMan assignment and resets AMS tray when the printer reports an empty slot.

### Data sources
- Print history DB default: `data/3d_printer_logs.db`
- Override via: `OPENSPOOLMAN_PRINT_HISTORY_DB`
- Mismatch log output: `logs/filament_mismatch.json` (now includes the detected color distance when a color mismatch occurs)

### Important operational note
If you change `OPENSPOOLMAN_BASE_URL`, NFC tags must be reconfigured.

---

## 6) SpoolMan integration contract (must remain stable)
### SpoolMan label workflow
- SpoolMan can print QR-code labels. When using them with OpenSpoolMan:
  - Set SpoolMan’s base URL to OpenSpoolMan **before** generating labels
  - Otherwise labels point back to SpoolMan, not OpenSpoolMan

### Required extra fields in SpoolMan
Agents must not “simplify away” these fields without an explicit migration plan.

Add these extra fields in SpoolMan:
- Filaments:
  - `type` (Choice)
  - `nozzle_temperature` (Integer Range)
  - `filament_id` (Text)
- Spools:
  - `tag` (Text)
  - `active_tray` (Text)

(Exact choice values are defined in the README; keep behavior compatible with existing installations.)

### Windows note (Bambu Studio)
Filament IDs can be sourced from Bambu Studio’s filament base directory (see README). Do not hardcode user paths; keep it documentation-only.

---

## 7) Filament matching rules (do not regress)
OpenSpoolMan matches SpoolMan spools to AMS tray metadata:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drndos/openspoolman](https://github.com/drndos/openspoolman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
