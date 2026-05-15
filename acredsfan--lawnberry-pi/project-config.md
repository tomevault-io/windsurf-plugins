---
trigger: always_on
description: Auto-generated from all feature plans. Last updated: 2026-04-20
---

# lawnberry Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-04-20

## Active Technologies
- Python 3.11 (backend), TypeScript + Vue 3 (frontend) + FastAPI, Uvicorn, Pydantic v2, websockets, Vue 3 + Vite, Pinia, Leaflet/Google Maps SDK (001-integrate-hardware-and)
- Python 3.11.x (backend), TypeScript (frontend - existing) + FastAPI, Uvicorn, Pydantic v2, asyncio, python-periphery, lgpio, pyserial, websockets, Vue 3 (existing), Vite, Pinia (002-complete-engineering-plan)
- SQLite for configuration/state, JSON logs with rotation, file-based cache for weather data (002-complete-engineering-plan)

## Project Structure
```
backend/
frontend/
tests/
```

## Commands

```bash
# Run unit tests only (fast, safe — no hardware I/O)
cd /home/pi/lawnberry && python -m pytest tests/unit/ -x -q -m "not hardware"

# Run a specific unit test file — NOTE: unit tests live in tests/unit/, NOT tests/
python -m pytest tests/unit/test_navigation_service.py -x -q
python -m pytest tests/unit/test_robohat_service.py -x -q

# Broader test run (may include integration tests — use the hardware filter)
python -m pytest tests/ -x -q -m "not hardware"

# Lint
ruff check backend/src

# Restart backend — NOTE: startup takes ~90 seconds (camera/AI init). Wait for health.
sudo systemctl restart lawnberry-backend
# Poll until the API responds (up to 2 min):
for i in $(seq 1 24); do sleep 5; curl -sf http://localhost:8081/api/v2/status && echo "UP" && break; echo "waiting... ($((i*5))s)"; done

# Tail backend logs
journalctl -u lawnberry-backend -f --no-pager
# OR
tail -f /home/pi/lawnberry/backend/backend.log
```

⚠️ **Test suite hang warning:** `python -m pytest tests/` without `-m "not hardware"` will hang
indefinitely because some tests block on hardware I/O (serial ports, I2C). Always filter or
set a per-test timeout. `pytest-timeout` is NOT installed; add `--timeout=N` only if it is.

**Test directory layout:**
- `tests/unit/` — fast unit tests (no hardware, always safe to run)
  - `tests/unit/test_navigation_service.py` — navigation unit tests (12 tests)
  - `tests/unit/test_robohat_service.py` — motor controller unit tests
- `tests/test_nav_coverage_patterns.py` — coverage/path algorithm tests (root level)
- `tests/test_nav_path_planner.py` — path planner tests (root level)
- `tests/test_mission_api.py` etc. — API integration tests (root level)

⚠️ **Wrong path anti-pattern:** `tests/test_navigation_service.py` does NOT exist — it is
at `tests/unit/test_navigation_service.py`. This mistake has caused repeated wasted runs.

## ⚠️ CRITICAL: Code Change Deployment Checklist

**After ANY commits to motor control, navigation, or driver code:**

This is **NOT optional** — changes to these systems do not become active until the backend is restarted and verified healthy. Skipping this step leads to debugging confusion (code is "fixed" but old code still running).

```bash
# 1. Clear Python bytecode cache (prevents stale .pyc from running)
find /home/pi/lawnberry -name "*.pyc" -delete
find /home/pi/lawnberry -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null || true

# 2. Restart the backend service
sudo systemctl restart lawnberry-backend

# 3. Poll until backend is healthy (startup takes ~90s, REQUIRED before testing)
for i in $(seq 1 24); do sleep 5; curl -sf http://localhost:8081/api/v2/status >/dev/null 2>&1 && echo "✓ Backend UP" && break; echo "waiting... ($((i*5))s)"; done

# 4. Verify full health (not just responsive)
curl -s http://localhost:8081/api/v2/status | jq '.safety_status'
```

**Do NOT claim a fix works until this checklist is complete.** If you get confused about whether a bug persists, first check: "Is the backend running my new code?"

---

## Motor Control Change Validation Pattern

⚠️ **CRITICAL: Two-Part Compensation System**

The LawnBerry motor control has **two interlocking compensation layers** that MUST be kept in sync:

1. **Navigation Service** (`backend/src/services/navigation_service.py:520-560`): Swaps left/right speed assignments
2. **RoboHAT Service** (`backend/src/services/robohat_service.py:835`): Inverts arcade mix sign

**If you modify ONLY ONE layer, the system breaks silently:**
- Remove arcade inversion but keep nav swap → joystick turns backward
- Remove nav swap but keep arcade inversion → navigation turns backward

**Validation Pattern (REQUIRED after any motor change):**

```bash
# Step 1: Unit tests first (fast, safe in SIM_MODE)
python -m pytest tests/unit/test_robohat_service.py tests/unit/test_navigation_service.py -xvs -m "not hardware"

# Step 2: If tests pass, restart backend (from checklist above)

# Step 3: Test joystick RIGHT TURN (left_motor must be POSITIVE, right_motor must be NEGATIVE)
curl -X POST http://localhost:8081/api/v2/control/drive \
  -H "Content-Type: application/json" \
  -d '{"throttle": 0.0, "turn": 0.5}'
# Expected: left_motor_speed > 0, right_motor_speed < 0

# Step 4: Test LEFT TURN (right_motor must be POSITIVE, left_motor must be NEGATIVE)
curl -X POST http://localhost:8081/api/v2/control/drive \
  -H "Content-Type: application/json" \
  -d '{"throttle": 0.0, "turn": -0.5}'
# Expected: right_motor_speed > 0, left_motor_speed < 0


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acredsfan/lawnberry_pi](https://github.com/acredsfan/lawnberry_pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
