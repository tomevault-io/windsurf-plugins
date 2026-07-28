---
trigger: always_on
description: PyPIStats.org is a Flask-based web application that provides analytics and visualization for Python package download statistics from PyPI (Python Package Index). It queries BigQuery public datasets, aggregates the data, and presents it through both a web interface and JSON API.
---

# PyPIStats.org - Python Package Download Analytics

## Overview
PyPIStats.org is a Flask-based web application that provides analytics and visualization for Python package download statistics from PyPI (Python Package Index). It queries BigQuery public datasets, aggregates the data, and presents it through both a web interface and JSON API.

## Architecture

### Core Technologies
- **Framework**: Flask (Python web framework)
- **Database**: PostgreSQL for storing aggregated statistics
- **Task Queue**: Celery with Redis for background processing
- **Data Source**: Google BigQuery (bigquery-public-data.pypi.file_downloads)
- **Visualization**: Plotly.js for interactive charts
- **Authentication**: GitHub OAuth for user features
- **Deployment**: Docker/Kubernetes support included

## Key Components

### 1. Data Pipeline (`pypistats/tasks/pypi.py`)
- **ETL Process**: Daily scheduled task (1am UTC) via Celery
- **BigQuery Integration**: Queries PyPI public dataset for download statistics
- **Data Categories**:
  - Overall downloads (with/without mirrors)
  - Python major versions (2, 3)
  - Python minor versions (2.7, 3.6, etc.)
  - Operating systems (Windows, Linux, Darwin, other)
- **Mirror Filtering**: Excludes downloads from known mirrors (bandersnatch, z3c.pypimirror, Artifactory, devpi)
- **Data Retention**: 180 days of historical data
- **Aggregation**: Creates __all__ package statistics for total ecosystem metrics

### 2. Database Models (`pypistats/models/download.py`)
- **OverallDownloadCount**: Total downloads per package per day
- **PythonMajorDownloadCount**: Downloads by Python major version
- **PythonMinorDownloadCount**: Downloads by Python minor version  
- **SystemDownloadCount**: Downloads by operating system
- **RecentDownloadCount**: Cached daily/weekly/monthly totals

### 3. Web Interface (`pypistats/views/general.py`)

#### Main Routes:
- `/` - Home page with package search
- `/packages/<package>` - Package statistics dashboard with interactive charts
- `/search/<package>` - Package search results
- `/top` - Top 20 packages by download count (day/week/month)
- `/about` - About page
- `/faqs` - Frequently asked questions

#### Features:
- Interactive time-series charts with date range selectors
- Download proportion visualizations
- PyPI metadata integration (dependencies, description)
- Customizable lookback periods (up to 180 days)

### 4. JSON API (`pypistats/views/api.py`)

#### Endpoints:
- `/api/packages/<package>/recent` - Recent download counts
  - Query params: `period` (day/week/month)
- `/api/packages/<package>/overall` - Overall download time series
  - Query params: `mirrors` (true/false)
- `/api/packages/<package>/python_major` - Downloads by Python major version
  - Query params: `version` (2/3)
- `/api/packages/<package>/python_minor` - Downloads by Python minor version
  - Query params: `version` (2.7/3.6/etc)
- `/api/packages/<package>/system` - Downloads by operating system
  - Query params: `os` (Windows/Linux/Darwin)

### 5. User Features (`pypistats/views/user.py`)
- GitHub OAuth integration
- Personal dashboard for package maintainers
- Track multiple packages in one view

## Data Flow

1. **Daily ETL Process**:
   - Celery scheduled task triggers at 1am UTC
   - Queries BigQuery for previous day's download data
   - Aggregates data by multiple dimensions
   - Stores results in PostgreSQL
   - Updates recent stats cache
   - Purges data older than 180 days
   - Runs VACUUM ANALYZE for database optimization

2. **Request Handling**:
   - User requests package stats via web or API
   - Flask queries PostgreSQL for aggregated data
   - Data formatted for Plotly.js visualization or JSON response
   - Results cached where appropriate

## Configuration

### Environment Variables:
- **Database**: `POSTGRESQL_*` (username, password, host, port, dbname)
- **Google Cloud**: `GOOGLE_*` (project_id, private_key, etc.)
- **GitHub OAuth**: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
- **Celery**: `CELERY_BROKER_URL` (Redis connection)
- **Flask**: `PYPISTATS_SECRET` (session secret key)

### Configuration Classes:
- `LocalConfig`: Local development
- `DevConfig`: Development environment
- `ProdConfig`: Production environment
- `TestConfig`: Testing environment

## Development

### Setup:
```bash
make pypistats  # Launch complete dev environment with docker-compose
```

### Project Structure:
```
pypistats/
├── application.py      # Flask app factory
├── config.py          # Configuration management
├── database.py        # Database utilities
├── extensions.py      # Flask extensions
├── models/           # SQLAlchemy models
├── tasks/            # Celery background tasks
├── views/            # Flask blueprints/routes
├── templates/        # Jinja2 HTML templates
├── static/           # CSS and static files
└── plots/            # Plotly chart configurations
```

### Dependencies:
- Python 3.7+
- Flask & extensions (SQLAlchemy, Migrate, Login, WTF, Limiter, HTTPAuth)
- Google Cloud BigQuery client
- Celery & Redis
- PostgreSQL (psycopg2)
- Requests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psf/pypistats.org](https://github.com/psf/pypistats.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
