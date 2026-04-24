---
trigger: always_on
description: > **All AI agents must conform to [CONTEXT.md](./CONTEXT.md)**
---

# VibeNVR AI Agent Instructions

> **All AI agents must conform to [CONTEXT.md](./CONTEXT.md)**

VibeNVR is a lightweight, modern Network Video Recorder (NVR) built with **Python (FastAPI)** backend and **React (Vite)** frontend. It leverages Docker for orchestration and FFmpeg for video processing.

## Development Environment

```bash
docker compose down && docker compose up -d --build && docker compose logs --tail 100    # Rebuild/Restart the dev environment
```

- Frontend: http://localhost:8080 (Nginx Production Build)
- Backend: http://localhost:5005 (FastAPI)
- API Docs: http://localhost:5005/docs

## Architecture Overview

### Backend (`backend/`)

```
backend/
├── main.py           # App entry point, CORS config, exception handlers
├── models.py         # SQLAlchemy Database Models
├── schemas.py        # Pydantic Schemas for validation
├── crud.py           # Database CRUD operations
├── database.py       # DB connection & SessionLocal
├── routers/          # API Routes (cameras, events, settings, etc.)
└── *_service.py      # Business logic (motion, storage, auth, etc.)
```

**Key patterns:**
- **Routers**: Handle HTTP request/response, validation, and auth.
- **Services**: Contain business logic. Called by routers.
- **CRUD**: Strictly database operations.
- **Dependency Injection**: Use `Depends(database.get_db)` and `Depends(auth_service.get_current_active_admin)`.

### Engine (`engine/`)

```
engine/
├── core.py               # CameraManager & Webhook Event Dispatcher
├── camera_thread.py      # Main loop, coordinates components & frame processing
├── stream_reader.py      # PyAV RTSP connection, IP ban protection, WS broadcast
├── recording_manager.py  # FFmpeg subprocess management (Passthrough, Transcoding, HW accel)
├── motion_detector.py    # MOG2 background subtraction and motion analysis
├── mask_handler.py       # JSON polygon parsing and frame masking
└── overlay_handler.py    # OSD text overlays on frames
```

**Key patterns:**
- **Modular Components**: `camera_thread.py` delegates IO and heavy processing to dedicated modules.
- **Resilience**: `stream_reader.py` implements smart backoff (401/403 -> 5m, Refused -> 1m) to avoid IP bans on standard consumer cameras (e.g. Tapo/TP-Link).
- **Graceful Degradation**: `recording_manager.py` falls back from Passthrough to software/HW encoding on failures.

### Frontend (`frontend/src/`)

```
src/
├── components/       # Reusable UI components
├── pages/            # Main page views (routed)
├── contexts/         # React Contexts (Auth, Toast, etc.)
├── assets/           # Static assets
├── App.jsx           # Main App component & Routing logic
└── main.jsx          # Entry point
```

**Key patterns:**
- **Components**: Functional React components with Hooks. Keep them small (<300 LOC preferred).
- **Modular Pages**: Large pages (e.g., `Cameras.jsx`) must be decomposed into a directory (e.g., `src/components/Cameras/`) with sub-components for modals and complex UI sections.
- **Styling**: TailwindCSS utility classes.
- **State Management**: React `useState` and `Context` for global state (Auth).
- **Icons**: `lucide-react` for all icons.

### Telemetry Worker (`vibenvr-telemetry-worker/`)

```
vibenvr-telemetry-worker/src/
├── index.js          # Main Cloudflare Worker entry point
├── api.js            # External API interactions
├── dashboard.js      # Status dashboard HTML rendering
├── ingest.js         # Telemetry data ingestion
├── security.js       # Security headers and CSP
└── assets.js         # Static asset delivery
```

**Key patterns:**
- **Cloudflare Worker**: Runs at the edge, requiring lightweight ES modules.
- **Strict Modularity**: Each file handles a single domain to respect the strict 1000 LOC policy.
- **Security Check**: Enforces CSP and API token requirements natively on the edge before requests hit the origin.

## Critical Patterns

### FastAPI Route Pattern

Routes must use `schemas` for validation and `Depends` for database sessions.

```python
# backend/routers/cameras.py (Pattern)
@router.post("", response_model=schemas.Camera)
def create_camera(
    camera: schemas.CameraCreate, 
    db: Session = Depends(database.get_db), 
    current_user: models.User = Depends(auth_service.get_current_active_admin)
):
    # Logic delegated to crud or service
    new_camera = crud.create_camera(db=db, camera=camera)
    motion_service.generate_motion_config(db)
    return new_camera
```

### React Authentication Pattern

The frontend uses `AuthContext` to manage the JWT token in memory. **Do not use `localStorage` for the token** — it is vulnerable to XSS. The token is persisted across page reloads via a `HttpOnly` cookie (`auth_token`) set by the backend on login.

All API calls — including media endpoints (`/cameras/{id}/frame`, `/cameras/{id}/stream`) — must use `Authorization: Bearer ${token}` in the request headers. The `media_token` HttpOnly cookie is maintained as a secondary fallback but should never be the sole authentication mechanism.

```jsx
// frontend/src/contexts/AuthContext.jsx (Pattern)
export const AuthProvider = ({ children }) => {
    // Token in memory ONLY — never in localStorage
    const [token, setToken] = useState(null);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spupuz/VibeNVR](https://github.com/spupuz/VibeNVR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
