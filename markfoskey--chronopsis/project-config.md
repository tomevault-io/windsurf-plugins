---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based timeline visualization application that displays scraped Wikipedia data (primarily British and American novels) in an interactive, zoomable timeline. Users can zoom with the mousewheel, pan by dragging, and click entries to view Wikipedia articles in an overlay.

## Repository Structure

The project consists of three main components:

- **client/**: Angular 16 frontend application
- **server/**: Flask backend server serving CSV data via REST API
- **fetcher/**: Python scripts for scraping Wikipedia data

## Development Commands

### Client (Angular)
Navigate to the `client/` directory for all Angular commands.

**Development server:**
```bash
cd client
npm install  # First time only
ng serve
```
Access at http://localhost:4200/

**Build:**
```bash
cd client
ng build  # Production build to dist/client
ng build --watch --configuration development  # Development build with watch
```

**Tests:**
```bash
cd client
ng test  # Runs Karma/Jasmine tests
```

### Server (Flask)
Navigate to the `server/` directory.

**Run server:**
```bash
cd server
python timeline_server.py
```

The server reads CSV files on startup and serves data through `/events` and `/allevents` endpoints. The hardcoded host/port is configured in `timeline_server.py:66`.

### Data Fetching
Navigate to the `fetcher/` directory.

**Scrape Wikipedia category:**
```bash
cd fetcher
python category_scraper.py "Category Name"
```

This recursively scrapes Wikipedia categories to extract novel metadata (title, author, publication date, genre, article length, etc.) into CSV files.

## Architecture

### Client Architecture

The Angular application is built around a single main component (`TimelineComponent`) that manages an HTML5 canvas-based visualization:

- **TimelineComponent** (`client/src/app/timeline/timeline.component.ts`): Core component handling all rendering, user interaction (zoom/pan/click), and data fetching. Uses direct canvas manipulation for performance.

- **Label** (`client/src/app/timeline/label.ts`): Represents individual timeline entries. Handles layout calculation, text wrapping, collision detection, and rendering. Calculates font size and importance from article length.

- **TimelineDataService** (`client/src/app/timeline/timeline-data.service.ts`): Angular service for fetching data from Flask backend. Supports both filtered queries (`/events`) and bulk loading (`/allevents`).

**Key rendering logic:**
- Year-based scaling: Maps years to pixel coordinates using `scaleAndShift()` and `rescale()` utilities
- Label placement: Labels sorted by importance (article length), positioned to avoid overlaps using `setY()` methods
- Alternating colored bands mark year intervals (dynamically calculated based on zoom level)
- Labels colored by author hash for visual grouping

**Interaction:**
- Mouse wheel: zoom in/out, centered on cursor position
- Click and drag: pan timeline horizontally and vertically
- Click label: opens Wikipedia article in left overlay iframe
- Touch events supported for mobile (incomplete pinch-to-zoom)

### Server Architecture

Simple Flask server (`server/timeline_server.py`) that:
1. Loads CSV files into memory on startup
2. Serves two endpoints:
   - `/events?start_date=X&end_date=Y&max_events=N`: Filtered events by date range, sorted by article length
   - `/allevents`: Returns all events sorted by article length
3. Uses `fetch_utils.py` from fetcher directory to parse dates

**Data format:** CSV files must contain columns like `title`, `author`, `year`, `day_in_year`, `genre`, `country`, `article_length`, `page_url`.

### Data Fetching Architecture

Wikipedia scraping utilities in `fetcher/`:

- **category_scraper.py**: Main scraper. Recursively traverses Wikipedia categories using MediaWiki API, extracts structured data from article infoboxes, and writes to CSV. Includes article length and inbound link counts as importance metrics.

- **fetch_utils.py**: Shared utilities for date parsing and extraction.

- **data_fetcher.py**: Additional data fetching utilities.

- **process_csv.py**: CSV post-processing scripts.

## Important Configuration Details

**Hardcoded network configuration:**
- Flask server host: `192.168.86.92:5500` (in `server/timeline_server.py:66`)
- Angular service baseUrl: `http://192.168.86.92:5500` (in `client/src/app/timeline/timeline-data.service.ts:30`)

Change these to `localhost` or appropriate addresses for your environment.

**Active data source:**
Currently only loading `BritishAndAmericanNovels.csv` (see `server/timeline_server.py:43`). The `historic_inventions.csv` is commented out.

## Data Flow

1. Wikipedia data scraped by `category_scraper.py` → CSV files in `server/`
2. Flask server loads CSV on startup → in-memory cache
3. Angular client requests data via HTTP → TimelineDataService
4. Data converted to Label objects → positioned and rendered on canvas
5. User interactions update visible year range → triggers re-fetch and re-render

## Known Issues & Incomplete Features


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markfoskey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
