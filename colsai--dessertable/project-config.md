---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DessertAble** is a Flask web application that finds and recommends dessert spots using Google Places API and DeepSeek AI. It features user authentication, favorites management, and an AI-powered ranking system that balances rating quality, distance, and newness.

## Core Commands

### Development
```bash
# Run the application locally
python run.py

# Or use the platform-specific scripts
# Windows: START_APP.bat
# Mac/Linux: ./START_APP.sh

# Run tests
pytest tests/

# Run specific test file
pytest tests/test_scoring_algorithm.py

# Run with verbose output
pytest -v
```

### Environment Setup
```bash
# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Mac/Linux)
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Production Deployment
```bash
# Run with Gunicorn (production)
gunicorn --workers 4 run:app

# Generate production SECRET_KEY
python -c "import secrets; print(secrets.token_hex(32))"
```

## Architecture Overview

### Application Factory Pattern
- `app/__init__.py` - Flask application factory using `create_app()`
- Configuration loaded from `config.py` based on `FLASK_ENV` environment variable
- Blueprint-based routing with single main blueprint (`main_bp`)

### Service Layer Architecture
The application uses a **service-oriented architecture** separating concerns:

1. **PlacesAPI** (`app/services/places_api.py`)
   - Wrapper around Google Maps Python client
   - Handles geocoding, nearby search, and place details
   - Built-in caching system (60-second TTL)
   - Uses Haversine formula for distance calculations

2. **DeepSeekService** (`app/services/deepseek_service.py`)
   - Generates 10-15 word AI descriptions from reviews
   - OpenAI-compatible API client pointing to DeepSeek
   - Gracefully degrades if API key not available

3. **RestaurantService** (`app/services/restaurant_service.py`)
   - Core business logic for restaurant search and ranking
   - **Progressive filter relaxation**: automatically widens search criteria if insufficient results
   - Composite scoring algorithm (see below)

### Database Layer
- `Database` class (`app/models/database.py`) supports **both SQLite and PostgreSQL**
- Auto-detects via `DATABASE_URL` env var: PostgreSQL in production, SQLite in development
- Database location (SQLite): `data/restaurants.db` (created automatically)
- Four main tables:
  - `searches` - Search history with parameters
  - `restaurants` - Cached restaurant data linked to searches
  - `users` - User accounts with password hashing
  - `favorites` - User-restaurant many-to-many with notes
- Key DB helper methods for cross-DB compatibility:
  - `get_connection()` - returns connection with proper row factory
  - `placeholder()` - returns `?` (SQLite) or `%s` (PostgreSQL)
  - `execute_insert()` - INSERT returning last row ID for either DB
  - `date_interval(days)` - DB-specific date arithmetic

### Authentication
- Flask-Login integration in `app/__init__.py`
- User model implements `UserMixin` interface
- Password hashing with werkzeug security utilities
- Session-based authentication with 7-day persistence
- **Invite-code-gated registration**: `INVITE_CODE` env var required to create accounts

### Admin Panel
- Separate session-based admin auth at `/admin/login` (not Flask-Login)
- Protected by `ADMIN_PASSWORD` env var; session key `is_admin`
- Dashboard at `/admin` (also `/admin/dashboard`) powered by `db.get_admin_analytics()` and `db.get_all_users(limit=50)`
- **Stat cards**: Total Users, Total Searches, Total Favorites, Avg Results — each with a +N last-7-days badge
- **Top Locations** (last 30 days) and **Popular Cuisines** panels with hover-slide rows
- **Recent Activity** table: last 10 searches with location, date, result count
- **User Management** table: all registered users with join date and favorite count
- **Search Activity** and **User Registrations** trend charts: bar-style rows for each of the last 30 days, normalized to max
- **Database Information** strip: cached restaurant count, total searches, total favorites
- **Export Data** button: downloads a CSV of the key metrics via `Blob` URL (client-side, no server call)
- Templates: `admin_login.html`, `admin_dashboard.html`

## Critical Algorithm: Composite Scoring

The **composite scoring algorithm** in `RestaurantService.calculate_composite_score()` is the heart of the ranking system:

**Weight Distribution:**
- **60%** - User's sort preference (rating OR distance)
- **15%** - Cuisine match bonus
- **20%** - Newness score (logarithmic decay based on review count)
- **5%** - Proximity bonus (always applied regardless of sort preference)

**Score Calculation:**
```
Total Score = Base Score (60) + Cuisine Bonus (0-15) + Newness (0-20) + Proximity (0-5)
Maximum Possible: 100 points
```

**Newness Formula:**
```
newness = max(0, 20 - (log10(review_count) * 8))
```
This creates logarithmic decay:
- 25 reviews ≈ 9 pts (newer establishments)
- 100 reviews ≈ 4 pts
- 500+ reviews ≈ 0 pts (established venues)

**Progressive Filter Relaxation:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Colsai/Dessertable](https://github.com/Colsai/Dessertable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
