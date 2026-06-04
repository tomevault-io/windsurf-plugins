---
trigger: always_on
description: **PROJECT**: Advanced Cortex usage analytics dashboard with React 19 + TypeScript frontend and Flask backend, designed for SPCS deployment.
---

<!-- markdown -->
# Snowflake Cortex Usage Analytics Dashboard - Cursor AI Rules

## SPCS React + Flask Application Template

**PROJECT**: Advanced Cortex usage analytics dashboard with React 19 + TypeScript frontend and Flask backend, designed for SPCS deployment.

**REFERENCE**: Based on proven SPCS patterns from https://github.com/sfc-gh-ujagtap/sun_valley_spcs

## Current Project Structure

```
cortex-usage-dashboard/
├── README.md                    # Comprehensive 1800+ line documentation
├── server.py                    # Flask server with Cortex analytics API endpoints
├── package.json                 # React 19 + latest dependencies
├── pyproject.toml              # Python dependencies with uv
├── uv.lock                     # Locked Python dependencies
├── tsconfig.json               # TypeScript configuration
├── Dockerfile                  # Multi-stage Docker build (React + Flask)
├── deploy.sh                   # 🚀 ONE-COMMAND deployment script with --update mode
├── docker-compose.yml          # Local development setup
├── env.example                 # Environment template
├── test-local-container.sh     # Container testing script
├── src/                        # React application source
│   ├── App.tsx                 # Main app with dashboard layout
│   ├── components/
│   │   ├── Dashboard.tsx       # Main analytics dashboard with Recharts
│   │   ├── ErrorBoundary.tsx   # Error handling component
│   │   └── ThemeSelector.tsx   # Theme switching component
│   ├── contexts/
│   │   └── ThemeContext.tsx    # Theme management context
│   ├── types/
│   │   └── index.ts           # TypeScript interfaces
│   └── themes.css             # Theme-aware styling
├── scripts/                   # Database setup scripts
│   ├── create_app_role.sql    # Creates APP_SPCS_ROLE with permissions
│   └── setup_database.sql     # Database setup for Cortex analytics
├── snowflake/                 # SPCS deployment files
│   ├── deploy.sql            # Service deployment with resource specs
│   ├── manage_service.sql    # Service management commands
│   ├── setup_image_repo.sql  # Image repository configuration
│   ├── snowflake_utils.py    # Snowflake connection utilities
│   └── snowflake_queries.py  # Cortex analytics queries
└── test/                     # Testing utilities
    ├── mock-spcs/token       # Mock SPCS token for local testing
    └── nginx.conf           # nginx configuration for container testing
```

### Core Architecture Principles

1. **Use Flat Project Structure** 
   - ✅ Use: Root-level React app with single `package.json`
   - Pattern: `src/`, `public/`, `build/`, `server.py` all at project root
   - Flask serves both API routes AND static React build files
   - Beware of `manifest.json` causing Content Security Policy errors in SPCS

2. **Port Strategy - Always 3002**
   - Use port 3002 consistently across ALL environments
   - Local development, Docker, SPCS service spec - all port 3002

3. **TypeScript Over JavaScript**
   - For React frontend, prefer TypeScript: convert `.js` to `.tsx`
   - Add `tsconfig.json` from https://github.com/sfc-gh-ujagtap/sun_valley_spcs

## Building Phase

### Flask Server Configuration

**Current Implementation** (`server.py`):
- ✅ **Snowpark Session Management** with dual authentication (SPCS OAuth + Local credentials)
- ✅ **Cortex Analytics API** endpoints for dashboard data
- ✅ **Enhanced Error Handling** with VPN connection checks
- ✅ **CORS enabled** for cross-origin requests
- ✅ **Static file serving** for React build files
- ✅ **Health monitoring** with detailed connection status

**API Endpoints**:
- `/api/health` - Health check with environment detection
- `/api/overall-costs` - Credits usage over time grouped by service
- `/api/costs-by-user` - Usage by user with totals and latest activity
- `/api/summary` - Aggregated totals, unique users, service breakdown, date range
- `/api/all-data` - Optimized bundle returning overall_costs, costs_by_user, summary + cache_info
- `/api/cache-info` - Cache timeout and last Snowflake pull time (no refresh)
- `/api/debug` - Environment and configuration diagnostics
- `/api/debug-users` - Unique user analysis for troubleshooting
- `/api/test-table` - Simple connectivity/schema probe

**Flask Integration Pattern** (CRITICAL):
```python
import sys
import os
sys.path.append(os.path.join(os.path.dirname(__file__), 'snowflake'))
from snowflake_utils import get_snowflake_session, is_running_in_spcs

from flask import Flask, jsonify, send_from_directory
import logging
import traceback

app = Flask(__name__, static_folder='build', static_url_path='')
logger = logging.getLogger(__name__)

def get_snowpark_session():
    """Get Snowpark session with enhanced logging"""
    try:
        logger.info("🔗 Attempting to create Snowpark session...")
        
        # Log environment variables (without sensitive values)
        env_vars = ['SNOWFLAKE_ACCOUNT', 'SNOWFLAKE_WAREHOUSE', 'SNOWFLAKE_DATABASE', 'SNOWFLAKE_SCHEMA', 'SNOWFLAKE_ROLE']
        missing_vars = []
        for var in env_vars:
            value = os.environ.get(var, 'NOT SET')
            logger.info(f"   {var}: {'SET' if value != 'NOT SET' else 'NOT SET'}")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfc-gh-jkang/cortex-cost-app-spcs](https://github.com/sfc-gh-jkang/cortex-cost-app-spcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
