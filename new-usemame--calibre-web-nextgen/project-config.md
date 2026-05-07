---
trigger: always_on
description: Calibre-Web Automated (CWA) is a fork of Calibre-Web that adds automated ebook processing, conversion, and management. It's a Flask web application running in Docker with Python 3.13, combining a modern web UI with Calibre's command-line tools for ebook manipulation.
---

# Copilot Instructions for Calibre-Web Automated

## Project Overview
Calibre-Web Automated (CWA) is a fork of Calibre-Web that adds automated ebook processing, conversion, and management. It's a Flask web application running in Docker with Python 3.13, combining a modern web UI with Calibre's command-line tools for ebook manipulation.

## Architecture

### Multi-Process Service Model (s6-overlay)
CWA uses **s6-overlay** for process supervision. Services are defined in `/root/etc/s6-overlay/s6-rc.d/`:
- **cwa-init**: One-time initialization (directory setup, permissions, Qt6 compatibility checks)
- **svc-calibre-web-automated**: Main Flask application
- **cwa-ingest-service**: File watcher that triggers ebook import/conversion via `ingest_processor.py`
- **metadata-change-detector**: Monitors `metadata.db` for changes to trigger cover/metadata enforcement
- **cwa-auto-zipper**: Daily compression of processed book backups
- **cwa-auto-library**: Automatic library detection and mounting

Services communicate via filesystem locks (`/tmp/*.lock`), SQLite databases, and status files (`/config/cwa_ingest_status`).

### Database Architecture
**Three separate SQLite databases** (never consolidate):
1. **`metadata.db`** (Calibre library) - Book metadata, managed by Calibre tools
2. **`app.db`** (Flask settings) - User accounts, permissions, CW configuration in `_Settings` table
3. **`cwa.db`** (CWA tracking) - Stats, settings, and audit logs. Schema in `scripts/cwa_schema.sql`, accessed via `CWA_DB` class

**WAL Mode**: Enabled by default on local disks for better concurrency. Disabled when `NETWORK_SHARE_MODE=true` due to NFS/SMB limitations.

### Flask Blueprint Organization
Core blueprints in `cps/main.py`:
- **CWA-specific**: `switch_theme`, `library_refresh`, `convert_library`, `epub_fixer`, `cwa_stats`, `cwa_settings`, `cwa_logs`, `profile_pictures`
- **Stock CW**: `web`, `opds`, `admin`, `editbook`, `shelf`, `kobo`, `oauth`, etc.

Each blueprint is a self-contained module in `cps/` (e.g., `cps/web.py`, `cps/cwa_functions.py`).

### Background Task System
**WorkerThread** (`cps/services/worker.py`) manages async tasks:
- Tasks inherit from `CalibreTask` base class
- Common tasks: `TaskConvert`, `TaskEmail`, `TaskBackupMetadata`, `TaskGenerateCoverThumbnails`
- Scheduled via APScheduler in `cps/schedule.py` (e.g., daily backups, thumbnail generation)
- Tasks stored in ImprovedQueue, status tracked with constants: `STAT_WAITING`, `STAT_STARTED`, `STAT_FINISH_SUCCESS`, etc.

### Automation Scripts
Python scripts in `/app/calibre-web-automated/scripts/`:
- **`ingest_processor.py`**: Core ingest logic - file validation, format conversion, Calibre import
- **`cover_enforcer.py`**: Applies UI metadata changes to actual ebook files using `ebook-meta`
- **`kindle_epub_fixer.py`**: EPUB sanitization for Kindle compatibility
- **`convert_library.py`**: Bulk format conversion across library
- **`cwa_db.py`**: Database wrapper class for CWA tracking database
- **`auto_library.py`**: Library auto-detection and mounting logic

Scripts use **filesystem locks** to prevent concurrent execution (e.g., `ingest_processor.lock`).

## Development Workflows

### Local Development Setup
1. **Build custom image**: Edit and run `build.sh` (prompts for repo dir, Docker Hub username, version)
2. **Development compose**: Use `docker-compose.yml.dev` with volume mounts for live-reload:
   ```yaml
   volumes:
     - ./cps:/app/calibre-web-automated/cps  # Live-edit Python
     - ./scripts:/app/calibre-web-automated/scripts
   ```
3. **Start container**: `docker compose -f docker-compose.yml.dev up -d`
4. **Default login**: admin/admin123 (change immediately)

### Testing Strategy
**No formal test suite exists** - manual testing workflow:
1. Drop test ebooks into `/cwa-book-ingest` bind
2. Monitor logs: `docker logs -f calibre-web-automated` or CWA Logs page in UI
3. Check CWA Stats page for import/conversion/enforcement counts
4. Verify `cwa.db` tables for audit trails: `cwa_import`, `cwa_conversions`, `cwa_enforcement`, `epub_fixes`

### Debugging Techniques
- **Service-specific logs**: Check `/config/log_archive/` for timestamped logs from each service
- **Lock file inspection**: Check `/tmp/*.lock` to identify stuck processes
- **Database queries**: Use `sqlite3 /config/cwa.db` to inspect stats/settings
- **Ingest queue**: Check `/config/cwa_ingest_retry_queue` for pending files
- **Status tracking**: Read `/config/cwa_ingest_status` for current ingest state

### Common Calibre Commands
CWA shells out to Calibre binaries (installed in `/app/calibre/`):
- **Import**: `calibredb add <file> --library-path=/calibre-library`
- **Convert**: `ebook-convert input.azw output.epub` (28 supported input formats)
- **Metadata**: `ebook-meta file.epub --set-cover=cover.jpg --title="New Title"`
- **Library check**: `calibredb list --library-path=/calibre-library --limit=1`

Always use `--library-path` flag explicitly. Never modify `metadata.db` directly.

## Code Conventions

### Error Handling Pattern
```python
from cps import logger

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [new-usemame/Calibre-Web-NextGen](https://github.com/new-usemame/Calibre-Web-NextGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
