---
trigger: always_on
description: | **Version** | 4.1.0 |
---

# ShotBase Project Context

## Quick Reference (Current State)

| Item | Value |
|---|---|
| **Version** | 4.1.0 |
| **Git Branch** | `main` |
| **Python Requirement** | ≥3.13.1 |
| **Default Host:Port** | 127.0.0.1:5001 |
| **Package Manager** | `uv` (do NOT use `pip install`) |
| **Linter** | ruff (config in pyproject.toml, line-length=120) |

---

## Project Overview
ShotBase is an application for managing AI-generated media assets including images, videos, prompts, and project notes. It supports structured organization, versioning, and annotation of generated stills and videos. The application provides a web interface for creating, managing, and organizing shots with drag-and-drop functionality.

Key features include:
- Shot management with versioned stills and videos
- Automatic organization of latest versions
- Prompt documentation and version history
- Shot reordering and archiving
- Display names for shots
- Asset promotion and captioning
- Thumbnail generation for images and videos (lazy, on-demand)
- First/last frame image variants per shot
- Alternative video asset (alt_video) for reference, upscales, or additional video variants
- Audio asset support (mp3, wav, ogg, flac, aac, m4a) with playback, promo, export, and search indexing
- Shot search modal with multi-token search, content/archive filters, and keyboard navigation
- Visual reorder modal with drag-and-drop grid, thumbnail switching, preview, and inline editing
- Project info management (title, version, description, tags)
- Export with metadata to markdown (including display name columns)
- Light/dark theme toggle
- Native folder picker for project creation
- Media modals with keyboard navigation (arrows)
- Gap-filling shot numbering (1–999)
- Sticky header on scroll
- Dynamic page title with project info and app version
- Collapsible archived section in TOC

---

## Technology Stack
- **Backend**: Python 3.13.1+, Flask
- **Frontend**: HTML/CSS/JavaScript (served by Flask)
- **Dependencies**: flask, flask-cors, pillow, python-dotenv
- **Build Tool**: uv (package manager and virtual environment tool)

---

## Development Tooling
This project uses `uv` as the primary tool for all Python development tasks including dependency management, running scripts, and virtual environment handling. 

**Important**: Avoid using `pip install` directly. Instead, use `uv` commands for all dependency management:
- Use `uv add package_name` to add new dependencies
- Use `uv remove package_name` to remove dependencies
- Use `uv sync` to install all project dependencies
- Use `uv run script_name.py` to run Python scripts

---

## Project Structure
```
shotbuddy/
├── app/                    # Main application code
│   ├── __init__.py         # Flask app factory
│   ├── utils.py            # Path sanitization, version reader
│   ├── routes/            # API routes (project_routes.py, shot_routes.py)
│   ├── services/          # Business logic
│   │   ├── shot_manager.py     # Core shot operations
│   │   ├── project_manager.py  # Project state, recent projects, info CRUD
│   │   ├── file_handler.py     # Uploads and asset processing
│   ├── config/            # Configuration (constants.py)
│   └── static/            # Static assets
│       ├── css/           # main.css, styles.css, search-modal.css, visual-reorder.css
│       ├── js/            # main.js, search-modal.js, visual-reorder.js, modal-behavior.js
│       └── icons/         # Favicons, PWA manifest, folder icon
├── shots/                  # Project data directory (created per project)
│   ├── wip/               # Work-in-progress shot folders
│   ├── latest_images/     # Latest image versions
│   ├── latest_videos/     # Latest video versions (regular, alt)
│   └── latest_audio/      # Latest audio versions
├── exports/               # Export output directory
├── run.py                 # Application entry point
├── shotbuddy.cfg          # Server configuration
├── pyproject.toml         # Project metadata and dependencies
├── requirements.txt       # Legacy dependencies list
└── uploads/               # Temporary upload directory
```

---

## Building and Running

### Prerequisites
- Python 3.13.1 or newer
- uv package manager

### Installation
1. Install uv: https://docs.astral.sh/uv/
2. Clone the repository
3. Create environment and install dependencies:
   ```bash
   uv sync
   ```
   
**Note**: This project uses `uv` for all dependency management. Do not use `pip install` directly as it may cause dependency conflicts or inconsistencies.

### Running the Application
1. Start the development server:
   ```bash
   uv run run.py
   ```
2. Open browser at http://127.0.0.1:5001/ (default)

### Configuration
Server settings can be configured in `shotbuddy.cfg`:
```ini
[server]
host = 0.0.0.0
port = 5001
```

Environment variables can override config file settings:
- `SHOTBUDDY_UPLOAD_FOLDER` - Upload directory (default: `uploads`)
- `SHOTBUDDY_HOST` - Server host (default: `127.0.0.1`)
- `SHOTBUDDY_PORT` - Server port (default: `5001`)
- `SHOTBUDDY_DEBUG` - Enable Flask debug mode (set to `1`)

---

## Development Conventions
- Uses Flask blueprints for route organization
- Project-scoped data management with ShotManager service
- JSON-based API responses with success/error structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taruma/shotbase](https://github.com/taruma/shotbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
