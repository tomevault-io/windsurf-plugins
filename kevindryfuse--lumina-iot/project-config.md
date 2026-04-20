---
trigger: always_on
description: ESP32-based LED strip controller designed for local/home server deployment. All services run in Docker Compose on a single machine.
---

# Lumina IoT - Local LED Controller

## Project Overview

ESP32-based LED strip controller designed for local/home server deployment. All services run in Docker Compose on a single machine.

**This is a fresh rewrite of the Azure-based led-controller project, simplified for local-only operation.**

## Architecture

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Browser   │────▶│   FastAPI + UI   │────▶│   Mosquitto     │
│             │     │   (HTMX-based)   │     │   MQTT Broker   │
└─────────────┘     └────────┬─────────┘     └────────┬────────┘
                             │                        │
                      PostgreSQL                      │
                      (users, state)                  │
                                              ┌───────▼───────┐
                                              │    ESP32      │
                                              │  LED Strip    │
                                              └───────────────┘
```

## Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| MQTT Broker | Mosquitto | Device communication |
| API + UI | FastAPI + HTMX + Jinja2 | Single app serves API and HTML |
| Database | PostgreSQL | User accounts, device state persistence |
| Auth | Session-based (bcrypt passwords) | Local user management |
| Containers | Docker Compose | One command deployment |

## Key Differences from Azure Version

| Feature | Azure Version | Local Version |
|---------|---------------|---------------|
| MQTT | Azure IoT Hub | Mosquitto |
| Auth | Azure Entra ID (OAuth) | PostgreSQL + bcrypt |
| UI | Streamlit (separate app) | HTMX (embedded in FastAPI) |
| State | Device Twins | PostgreSQL |
| Deploy | Terraform + Container Apps | Docker Compose |
| Cost | ~$5-10/month | Free (self-hosted) |

---

## Project Structure

```
lumina-iot/
├── CLAUDE.md              # This file - project docs
├── docker-compose.yml     # All services
├── .env.example           # Environment template
├── api/
│   ├── Dockerfile
│   ├── pyproject.toml
│   └── src/
│       ├── main.py        # FastAPI app
│       ├── auth.py        # User auth (bcrypt, sessions)
│       ├── mqtt.py        # Mosquitto client
│       ├── db.py          # PostgreSQL models
│       └── templates/     # Jinja2 + HTMX templates
│           ├── base.html
│           ├── login.html
│           ├── dashboard.html
│           └── partials/
│               └── device_card.html
├── mosquitto/
│   └── mosquitto.conf     # Broker config
└── scripts/
    └── create_user.py     # CLI to add users
```

**Note:** ESP32 firmware is in a separate repo: [lumina-esp32](../lumina-esp32)

---

## Docker Compose Services

```yaml
services:
  mosquitto:    # MQTT broker, port 1883
  postgres:     # Database, port 5432
  api:          # FastAPI + UI, port 8000
```

All on the same Docker network (`lumina-net`).

---

## Implementation Plan

### Phase 1: Foundation ✅
- [x] Create project structure
- [x] Docker Compose with Mosquitto + PostgreSQL
- [x] Basic FastAPI app with health check
- [x] Database models (users, devices, device_state)

### Phase 2: Authentication ✅
- [x] User model with bcrypt password hashing
- [x] Login/logout endpoints
- [x] Session middleware (cookie-based)
- [x] create_user.py CLI script

### Phase 3: MQTT Integration ✅
- [x] Mosquitto config (anonymous for devices)
- [x] FastAPI MQTT client (subscribe to device topics)
- [x] Device registration via MQTT announcements
- [x] Command publishing (color, brightness, effect)

### Phase 4: HTMX UI ✅
- [x] Base template with Tailwind CSS
- [x] Login page
- [x] Dashboard with device cards
- [x] Color picker (HTMX swap)
- [x] Brightness slider
- [x] Effect buttons
- [ ] Real-time state updates (SSE or polling) — optional enhancement

### Phase 5: ESP32 ✅
- [x] Simplified local-only version (moved to separate repo: lumina-esp32)
- [x] Removed all Azure code
- [x] Auto-generated device ID from chip ID
- [ ] Test with Docker Compose stack — needs testing

### Phase 6: Polish
- [x] State persistence to PostgreSQL
- [x] Device naming UI (friendly names) - hover device name to edit
- [ ] Error handling and logging improvements
- [x] README with deployment instructions

---

## Database Schema

### users
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### devices
```sql
CREATE TABLE devices (
    id SERIAL PRIMARY KEY,
    device_id VARCHAR(100) UNIQUE NOT NULL,
    friendly_name VARCHAR(100),
    device_type VARCHAR(50) DEFAULT 'led_strip',
    last_seen TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### device_state
```sql
CREATE TABLE device_state (
    id SERIAL PRIMARY KEY,
    device_id VARCHAR(100) UNIQUE NOT NULL REFERENCES devices(device_id),
    brightness INTEGER DEFAULT 100,
    color_r INTEGER DEFAULT 255,
    color_g INTEGER DEFAULT 255,
    color_b INTEGER DEFAULT 255,
    effect VARCHAR(50) DEFAULT 'none',
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## HTMX Patterns

### Color Picker Example
```html
<form hx-post="/devices/esp32-01/color"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KevinDryfuse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
