---
trigger: always_on
description: LiteCron is a lightweight Docker-based Python cron job scheduler with WebUI and notification support.
---

# LiteCron - Agent Development Guide

LiteCron is a lightweight Docker-based Python cron job scheduler with WebUI and notification support.

## Quick Reference

### Essential Commands

```bash
# Interactive menu (recommended)
python manage.py

# Container management
python manage.py start|stop|restart|status|logs|shell|build|update

# Task management
python manage.py list                    # List scheduled tasks
python manage.py run <TaskName>          # Run specific task (in container)
python manage.py run --all               # Run all enabled tasks (in container)
python manage.py run-local <TaskName>    # Run task locally (no Docker required)
python manage.py run-local --all         # Run all enabled tasks locally
python manage.py tasklogs                # View task logs
python manage.py validate                # Validate config.yml

# Notifications
python manage.py notify "message"        # Send test notification
python manage.py notify "message" -l     # With recent logs
python manage.py notify "message" -l -n 30  # With 30 lines of logs

# Maintenance
python manage.py clean                   # Clean logs older than 7 days
python manage.py build [--no-cache]      # Build Docker image
python manage.py build v1.0.0            # Build with custom tag
```

### Local Development

```bash
pip install -r requirements.txt
python src/webapp.py                     # Run WebUI locally (port 5000)
python tasks/ikuuu.py                    # Test task script directly
```

## Architecture

### Directory Structure

```
lite-cron/
├── manage.py              # Main management script (interactive + CLI)
├── config.yml             # Task configuration (GITIGNORED - user-specific)
├── config.example.yml     # Configuration template
├── compose.yml            # Docker compose (GITIGNORED)
├── compose.example.yml    # Docker compose template
├── Dockerfile             # Python 3.11-slim based
├── requirements.txt       # Dependencies: PyYAML, requests, Flask, croniter, curl_cffi
├── src/                   # Core source (runs in container as /app/)
│   ├── entrypoint.sh      # Container entrypoint - starts cron + WebUI
│   ├── task_wrapper.py    # Task execution wrapper with notifications
│   ├── webapp.py          # Flask WebUI (port 5000)
│   ├── notify.py          # Notification module (Webhook + NTFY)
│   ├── make_cron.py       # Generates crontab from config.yml
│   ├── make_env.py        # Generates .env from config.yml
│   ├── logger.py          # Python logging (file + stdout)
│   ├── logger.sh          # Shell logging (for entrypoint.sh)
│   ├── template/          # Flask templates
│   └── static/            # Static assets
├── tasks/                 # Task scripts (user-facing)
│   ├── ikuuu.py, pttime.py, smzdm.py, tieba.py, fnclub.py,
│   ├── aliyunpan.py, bilibili.py, v2ex.py, nodeseek.py, zhutix.py
│   └── README.md          # Task documentation
├── logs/                  # Runtime logs (date-based: YYYYMMDD.log)
└── data/                  # Persistent data
```

### Key Architecture Points

1. **Container Path Mapping**: `src/` maps to `/app/` in container. Task scripts reference paths relative to `/app/`.

2. **Environment Variable Flow**:
   - `config.yml` → `make_env.py` → `/app/.env` → loaded by `task_wrapper.py`
   - Task env vars are prefixed: `TaskName_KEY` in container → `KEY` in task script
   - Global env vars set via `global_env` in config.yml

3. **Task Execution Chain**:
   - Cron triggers → `task_wrapper.py <TaskName> <script_path>` → actual script
   - Wrapper handles: env setup, logging, timing, failure notifications
   - Execution modes: `cron` (default), `webui`, `cli` (via `LITECRON_EXEC_MODE`)

4. **Notification System**:
   - Configured in `config.yml` under `notify` section
   - Supports Webhook (JSON/form/text) and NTFY
   - Auto-triggers on task failure (configurable for success too)
   - `notify.py` can be called standalone for testing

## Task Script Conventions

### Standard Template

```python
#!/usr/bin/env python3
"""
Task Name - Brief description

Environment Variables:
- VAR_NAME: Description (required)
- OPTIONAL_VAR: Description (optional)
"""
import os
import sys

# Import project logger (REQUIRED - don't use standard logging)
sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "src"))
from logger import log_info, log_success, log_error, log_warning, log_debug

# Read env vars
MY_VAR = os.environ.get("MY_VAR")


def main() -> int:
    """Main function - returns 0 for success, 1 for failure"""
    log_info("🚀 Task started")
    
    try:
        # Task logic here
        result = do_something()
        if result:
            log_success("✅ Task completed")
            return 0
        else:
            log_warning("⚠️ Task failed")
            return 1
    except Exception as e:
        log_error(f"❌ Error: {str(e)}")
        return 1
    finally:
        log_info("🏁 Task finished")


if __name__ == "__main__":
    sys.exit(main())
```

### Critical Rules for Task Scripts

1. **Always use project logger**: `from logger import log_info, ...` - NOT `import logging`
2. **Return codes**: `0` = success, `1` = failure (wrapper uses this for notifications)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdone/lite-cron](https://github.com/pdone/lite-cron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
