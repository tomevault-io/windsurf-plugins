---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack Croatian events platform ("Kružna Karta Hrvatska") built as a monorepo with:
- **Frontend**: React 19 + TypeScript + Tailwind CSS + Vite (located in `frontend-new/`)
- **Backend**: FastAPI + Python + PostgreSQL with advanced geocoding and scraping capabilities
- **Infrastructure**: Docker with comprehensive development tooling

The platform aggregates events from 14+ Croatian websites through intelligent web scraping, provides real-time geocoding for venues, and features an advanced map system with clustering capabilities powered by Mapbox GL JS.

## Key Architecture

### Monorepo Structure
- `frontend-new/` - React 19 TypeScript application with advanced mapping
- `backend/` - FastAPI Python application with geocoding and scraping services
- `docker/` - Docker configurations and infrastructure
- Root-level `Makefile` with 40+ development commands

### Technology Stack
- **Frontend**: React 19, TypeScript 5.8, Tailwind CSS 3.4, Vite 7, Mapbox GL JS 3.13, TanStack Query 5
- **Backend**: FastAPI, SQLAlchemy 2.0, Alembic, Celery, Redis, Playwright, Pydantic 2.4+
- **Database**: PostgreSQL with Redis for caching/sessions, advanced geocoding tables
- **Deployment**: Docker Compose, Nginx reverse proxy, uv package management

### Advanced Features
- **Map System**: Mapbox GL JS with intelligent clustering, real-time filtering, and performance optimization
- **Geocoding Service**: Real-time venue geocoding with Croatian geographic database and fallback systems
- **Scraping System**: 14+ specialized scrapers (Entrio, InfoZagreb, Visit sites, etc.) with enhanced error handling
- **Type-Safe Configuration**: Pydantic-based configuration management with environment variable expansion
- **Performance Optimization**: Throttled map updates, marker batching, clustering transitions

## Development Commands

The project uses a comprehensive Makefile with 40+ commands. Key commands:

### Quick Start
```bash
make setup           # Initial project setup
make dev            # Start full development environment
make frontend-dev   # Frontend only
make backend-dev    # Backend only
```

### Daily Development
```bash
make logs           # View all logs
make frontend-logs  # Frontend logs only
make backend-logs   # Backend logs only
make db-shell      # PostgreSQL shell access
make redis-shell   # Redis CLI access
```

### Code Quality
```bash
make lint          # Run all linters
make test          # Run all tests
make frontend-test # Frontend tests only
make backend-test  # Backend tests with coverage
```

### Database Management
```bash
make migrate       # Run database migrations
make migration     # Create new migration
make db-reset     # Reset database with sample data
```

## Key Business Logic

### Enhanced Scraping System
**Multi-Source Event Scraping**:
- **Base Architecture**: Abstract base scraper with common functionality for retry logic, date parsing, and text cleaning
- **BrightData Integration**: Proxy support with scraping browser for JavaScript-heavy sites
- **Playwright Support**: Dynamic content scraping for Vue.js/React applications
- **Specialized Scrapers**: 14+ Croatian event sources including Croatia.hr, Entrio, Info Zagreb, city tourism sites
- **Data Transformation**: Croatian date/time parsing, location extraction, and event normalization
- **Deduplication**: Hash-based duplicate detection and event merging

**Supported Event Sources**:
- `croatia_scraper.py` - Croatia.hr official tourism events (Vue.js-based)
- `entrio_scraper.py` - Entrio ticketing platform
- `infozagreb_scraper.py` - Zagreb city events
- City tourism sites: Split, Rijeka, Dubrovnik, Karlovac, Opatija, Varaždin, Vukovar, Zadar
- `ulaznice_scraper.py` - Ulaznice.hr ticketing platform

### Advanced Geocoding and Location Services
**Real-time Geocoding Service**:
- **Multi-Provider Strategy**: Mapbox API primary, Nominatim (OpenStreetMap) fallback
- **Croatian Geographic Database**: Built-in database of 70+ Croatian cities, regions, and landmarks with coordinates
- **Venue Coordinate Caching**: Database caching of geocoded venues with 30-day refresh cycle
- **Fallback Strategies**: Croatian city center fallback, Zagreb coordinates as ultimate fallback
- **Address Enhancement**: Croatian address pattern recognition and street-level geocoding
- **Batch Processing**: Concurrent geocoding with rate limiting and retry logic

**Croatian Geographic Coverage**:
- Major cities: Zagreb, Split, Rijeka, Osijek, Zadar, Pula, Dubrovnik
- Tourist destinations: Hvar, Korčula, Krk, Rovinj, Opatija, Makarska
- Regional centers and counties with population-based confidence scoring
- Fuzzy matching for Croatian diacritics (č, ć, š, ž, đ)

### Configuration System
**Type-Safe Configuration Management**:
- **Pydantic Settings**: Full type validation with automatic environment variable binding
- **YAML Configuration**: `backend/config.yaml` with environment variable expansion (`${VAR:default}`)
- **Component-Based Structure**: Database, Redis, API, Auth, Scraping, Services, Monitoring configs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JS1240) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
