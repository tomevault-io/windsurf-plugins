---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Shoebox - AI Collaboration Guide

## Project Overview

**Shoebox** is a self-hosted video organization and preservation application for creative workflows. Unlike photo management services, it focuses on **video cataloging** with export-oriented features for use in external video editors.

**Important**: This project is in active development and is not yet safe for production use. Data models may change.

## Technology Stack

- **Backend**: Rust 2021, Axum 0.8, Tokio, SQLx 0.8
- **Frontend**: React 18, TypeScript, Vite 6, Chakra UI 2
- **Database**: PostgreSQL (primary), SQLite (in Cargo.toml but not officially supported)
- **Media Processing**: FFmpeg (system binary via `std::process::Command`)
- **Deployment**: Docker, Docker Compose, Helm

## Development Workflows

### Backend
```bash
cargo run                      # Run backend (localhost:3000)
cargo build --release          # Build release binary
cargo clippy --all-features    # Linting
cargo test --all-features      # Run tests
```

### Frontend
```bash
cd frontend
yarn install                   # Install dependencies
yarn dev                       # Dev server (localhost:5173)
yarn build                     # Production build
```

### Full Stack
```bash
docker-compose up -d           # Run full stack with containers
```

## Key Configuration

Critical environment variables (loaded via `dotenv`):
- `SERVER_HOST` - Bind host (default: `0.0.0.0`)
- `SERVER_PORT` - Server port (default: `3000`)
- `DATABASE_URL` - PostgreSQL connection string (e.g., `postgresql://user:pass@host/db`)
- `MEDIA_SOURCE_PATHS` - Semicolon-separated video directories to scan
- `THUMBNAIL_PATH` - Thumbnail storage directory
- `EXPORT_BASE_PATH` - Export destination directory
- `FRONTEND_PATH` - Frontend dist directory (auto-detected: `/app/frontend/dist` or `frontend/dist`)

## Architecture Notes

### Backend Structure (`/src/`)

```
src/
├── main.rs          # Entry point, Axum router setup, static file serving
├── config.rs        # Config struct with FromEnv, media path parsing
├── db.rs            # PostgreSQL pool init, sqlx migrations
├── error.rs         # AppError enum with IntoResponse implementation
├── models.rs        # Model module re-exports
├── models/
│   ├── video.rs     # Video, VideoWithMetadata, Create/Update/BulkUpdate DTOs
│   ├── tag.rs       # Tag model
│   ├── person.rs    # Person model
│   └── shoebox.rs   # Shoebox model (folder organization)
├── routes/          # Axum route modules
│   ├── mod.rs       # API router composition
│   ├── video.rs     # /api/videos endpoints
│   ├── tag.rs       # /api/tags endpoints
│   ├── person.rs    # /api/people endpoints
│   ├── location.rs  # /api/locations endpoints
│   ├── event.rs     # /api/events endpoints
│   ├── shoebox.rs   # /api/shoeboxes endpoints
│   ├── scan.rs      # /api/scan endpoints
│   ├── export.rs    # /api/export endpoints
│   ├── system.rs    # /api/system endpoints
│   └── media.rs     # /media video serving
├── services/        # Business logic layer
│   ├── mod.rs       # AppState (Pool<Postgres>, Config, ScanStatus)
│   ├── scanner.rs   # Video directory scanning, duplicate detection
│   ├── thumbnail.rs # FFmpeg thumbnail generation
│   ├── video.rs     # Video CRUD, search, metadata queries
│   ├── tag.rs       # Tag management
│   ├── person.rs    # Person management
│   ├── location.rs  # Location management
│   ├── event.rs     # Event management
│   ├── shoebox.rs   # Shoebox (folder) management
│   └── export.rs    # Video export to external editors
└── utils/
    ├── mod.rs
    └── file.rs      # File utility functions
```

### Frontend Structure (`/frontend/src/`)

```
frontend/src/
├── main.tsx         # React entry point
├── App.tsx          # Router setup, page routing
├── config.ts        # API base URL configuration
├── api/
│   └── client.ts    # Axios instance with base URL
├── contexts/
│   └── ScanContext.tsx  # Scan state management
├── components/
│   └── SearchFilters.tsx  # Video search UI
└── pages/
    ├── HomePage.tsx
    ├── UnreviewedPage.tsx
    ├── VideoDetailPage.tsx
    ├── RatedVideosTimelinePage.tsx
    ├── BulkEditPage.tsx
    ├── ExportPage.tsx
    ├── ManagementPage.tsx
    └── SystemInfoPage.tsx
```

### State Management

Backend `AppState` (clonable via Arc pattern):
```rust
pub struct AppState {
    pub db: Pool<Postgres>,
    pub config: Config,
    pub scan_status: Arc<RwLock<ScanStatus>>,
}
```

## Database Schema

### Core Tables
- `videos` - Video metadata (id, file_path, file_name, title, description, rating 1-5, duration, etc.)
- `tags` - Tag definitions (id, name UNIQUE)
- `people` - Person definitions (id, name UNIQUE)
- `locations` - Location definitions (id, name UNIQUE)
- `events` - Event definitions (id, name UNIQUE)
- `shoeboxes` - Folder organization (id, name UNIQUE, path)

### Junction Tables
- `video_tags(video_id, tag_id)` - Many-to-many video-tag relationships
- `video_people(video_id, person_id)` - Many-to-many video-person relationships
- `video_locations(video_id, location_id)` - Many-to-many video-location relationships

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devclef/shoebox](https://github.com/devclef/shoebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
