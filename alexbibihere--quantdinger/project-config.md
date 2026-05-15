---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QuantDinger is a quantitative trading monitoring system based on the HAMA (Hull Moving Average) technical indicator. The system uses browser automation (Playwright/Brave) to scrape TradingView charts and local OCR (RapidOCR) to extract HAMA indicator data.

**Tech Stack:**
- **Backend**: Python 3.11+ with Flask 2.3, Playwright 1.40, RapidOCR 1.3, SQLAlchemy 2.0
- **Frontend**: Vue 2.6.14, Ant Design Vue 1.7.8, ECharts 6.0, Axios
- **Database**: SQLite (primary), Redis (optional cache)

## Development Commands

### Backend (Python)

```bash
cd backend_api_python

# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers
playwright install chromium

# Run development server (port 5000)
python run.py

# Run with gunicorn (production)
gunicorn -c gunicorn_config.py "run:app"
```

### Frontend (Vue.js)

```bash
cd quantdinger_vue

# Install dependencies
npm install

# Run development server (port 8000)
npm run serve

# Build for production
npm run build

# Lint code
npm run lint
```

### Full Stack Development

```bash
# Terminal 1:
cd backend_api_python && python run.py

# Terminal 2:
cd quantdinger_vue && npm run serve
```

## Architecture Overview

```
QuantDinger/
├── backend_api_python/          # Python Flask backend
│   ├── app/
│   │   ├── __init__.py          # Application factory with singletons
│   │   ├── routes/              # API blueprints (endpoints)
│   │   ├── services/            # Business logic layer
│   │   ├── config/              # Configuration classes
│   │   ├── data/                # SQLite databases
│   │   └── utils/               # Utilities (logger, SSE, etc.)
│   └── run.py                   # Flask entry point
│
├── quantdinger_vue/             # Vue.js frontend
│   ├── src/
│   │   ├── views/               # Page components
│   │   ├── mixins/              # Vue mixins (realtimePrice, etc.)
│   │   └── utils/               # Utilities (request, SSE, etc.)
│   └── package.json
│
└── docs/                        # Additional documentation
    ├── BRAVE_MONITOR_LOGIC.md   # Brave monitoring details
    └── TECH_STACK.md            # Complete dependency list
```

## Backend Architecture

### Application Factory Pattern

The Flask app uses the application factory pattern ([`app/__init__.py`](backend_api_python/app/__init__.py)). Key singletons initialized at startup:
- `_redis_client` - Redis connection (optional)
- `_hama_scheduler` - APScheduler for periodic HAMA data fetching
- `_hama_brave_monitor` - Browser automation monitor
- `_trading_executor` - Strategy execution engine
- `_pending_order_worker` - Order dispatch worker
- `_reflection_worker` - AI reflection verification

### Key Services

1. **HAMA Brave Monitor** ([`services/hama_brave_monitor.py`](backend_api_python/app/services/hama_brave_monitor.py))
   - Controls Brave/Chromium browser in headless mode
   - Navigates to TradingView charts, captures screenshots
   - Parallel monitoring with `monitor_batch_parallel()`
   - SQLite + Redis dual-layer caching

2. **OCR Extractor** ([`services/hama_ocr_extractor.py`](backend_api_python/app/services/hama_ocr_extractor.py))
   - Uses RapidOCR for local text recognition (no API keys needed)
   - Extracts: HAMA value, color, trend, MA status, Bollinger status
   - Clip coordinates configurable for screenshot region

3. **Trading Executor** ([`services/trading_executor.py`](backend_api_python/app/services/trading_executor.py))
   - Manages trading strategy lifecycle (start/stop)
   - Handles indicator-based strategies
   - Simulated trading (paper mode)

4. **Monitor Worker** ([`services/hama_monitor_worker.py`](backend_api_python/app/services/hama_monitor_worker.py))
   - Background thread continuously monitoring configured symbols
   - Detects HAMA crossover signals (金叉/死叉)
   - Sends email notifications on signal detection

### API Routes Structure

Routes are organized by feature in [`app/routes/`](backend_api_python/app/routes/):
- `dashboard.py` - KPI metrics, trades, positions
- `hama_market.py` - HAMA watchlist, symbol data, signals
- `hama_monitor.py` - Smart monitoring control
- `tradingview_scanner.py` - Top gainers, screenshots
- `trading_assistant.py` - AI decisions, positions
- `settings.py` - Configuration management

### Environment Configuration

Create [`backend_api_python/.env`](backend_api_python/.env):

```env
# Flask
FLASK_APP=run.py
FLASK_ENV=development
SECRET_KEY=your-secret-key

# TradingView
TRADINGVIEW_URL=https://cn.tradingview.com/chart/U1FY2qxO/

# Brave Monitor
BRAVE_MONITOR_ENABLED=true
BRAVE_MONITOR_CACHE_TTL=900
BRAVE_MONITOR_BROWSER_TYPE=brave
BRAVE_PATH=C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe

# HAMA Scheduler
HAMA_SCHEDULER_ENABLED=true
HAMA_SCHEDULER_AUTO_START=true

# Redis (optional)
REDIS_ENABLED=false
REDIS_HOST=localhost
REDIS_PORT=6379

# Monitoring
HAMA_DEMO_MODE=false
```

### TradingView Cookie Configuration (Optional)

For auto-login, create [`backend_api_python/tradingview_cookies.json`](backend_api_python/tradingview_cookies.json):

```json
{
  "cookies": [
    {
      "name": "sessionid",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexbibihere/QuantDinger](https://github.com/alexbibihere/QuantDinger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
