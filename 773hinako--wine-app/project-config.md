---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PWA-focused wine tasting record app** with offline support.

This project has been restructured to focus exclusively on **Progressive Web App (PWA)** development. Previous iOS and Standalone versions have been archived for reference.

Core technology: Vanilla JavaScript + IndexedDB for complete offline functionality.

## Project Structure

```
wine/
├── src/                    # Source code
│   ├── js/                # JavaScript modules
│   │   ├── app.js        # Main application logic & UI
│   │   ├── db.js         # IndexedDB wrapper (WineDB class)
│   │   ├── features.js   # Extended features (sort, filter, favorites, stats, dark mode)
│   │   └── auto-backup.js # Auto-backup to LocalStorage
│   ├── css/              # Stylesheets
│   │   └── styles.css    # Main stylesheet with dark mode support
│   └── workers/          # Service Worker
│       └── service-worker.js
│
├── public/               # Static files (deployed as-is)
│   ├── index.html       # Single-page app HTML
│   ├── manifest.json    # PWA manifest
│   └── icons/           # PWA icons
│       ├── icon-192.png
│       └── icon-512.png
│
├── archive/              # Archived versions (iOS, Standalone)
├── docs/                 # Documentation
├── scripts/              # Development scripts
├── netlify.toml         # Netlify deployment config
├── GIT-GUIDE.md         # Git workflow documentation
└── DATA-SAFETY.md       # Data persistence documentation
```

## Common Commands

### Development Server

```bash
# Quick start with provided scripts
./scripts/start-server.bat        # Windows
./scripts/start-server.sh         # Mac/Linux

# Manual start
python -m http.server 8000
```

### Icon Generation

```bash
# PWA icons (192x192, 512x512)
python scripts/generate-icons.py

# Browser-based icon generation
# Open scripts/create-icons.html in browser
```

## Architecture

### Screen Management

The app uses a single-page architecture with three screens managed by JavaScript:
- **Home Screen** (`#home-screen`): Wine list with search functionality
- **Edit Screen** (`#edit-screen`): Add/edit wine form with camera integration
- **Detail Screen** (`#detail-screen`): View full wine details

Screen transitions are handled by `showScreen()` function in [src/js/app.js](src/js/app.js).

### Module Architecture

The app is split into focused modules:

1. **[src/js/db.js](src/js/db.js)** - Data persistence layer
   - `WineDB` class wraps IndexedDB operations
   - Handles CRUD operations and search
   - Export/import functionality

2. **[src/js/features.js](src/js/features.js)** - Extended features
   - Sort/filter utilities (`sortWines`, `filterWines`)
   - Favorite toggle (`toggleFavorite`)
   - Statistics calculation (`calculateStatistics`)
   - Dark mode (`initDarkMode`, `toggleDarkMode`)
   - Tutorial system (`showTutorial`)
   - Swipe gestures (`initSwipeGestures`)

3. **[src/js/auto-backup.js](src/js/auto-backup.js)** - Data safety
   - Auto-backup to LocalStorage every 5 minutes
   - Maintains 3 generations of backups
   - Backup on page unload

4. **[src/js/app.js](src/js/app.js)** - Application controller
   - Screen navigation (`showScreen`)
   - Event handling
   - UI rendering and updates
   - Coordinates other modules

### Data Flow

1. User interactions → Event handlers in [src/js/app.js](src/js/app.js)
2. App logic → IndexedDB operations via [src/js/db.js](src/js/db.js) WineDB class
3. Data stored in IndexedDB + auto-backup to LocalStorage
4. UI updated with new data
5. Service Worker caches app files for offline use

### IndexedDB Schema

**Object Store**: `wines`
- **Key**: `id` (auto-increment)
- **Indexes**: `name`, `region`, `variety`, `date`

**Wine Object Structure**:
```javascript
{
  id: number,              // Auto-generated
  name: string,           // Required
  producer: string,
  region: string,
  variety: string,
  vintage: number,
  date: string,           // ISO date format
  rating: number,         // 0-5
  photo: string,          // Base64 data URL (compressed, max 1200px)
  thumbnail: string,      // Base64 thumbnail (400px, for list view)
  favorite: boolean,      // Favorite flag
  tasting: {              // Structured tasting notes
    wineType: string,     // 'red', 'white', 'rose', 'sparkling'
    appearanceColor: string,
    aromas: string[],
    firstAroma: string,   // Primary aroma (from grape variety)
    secondAroma: string,  // Secondary aroma (from fermentation)
    thirdAroma: string,   // Tertiary aroma/bouquet (from aging)
    oakIntensity: string, // Oak/barrel influence intensity
    sweetness: string,
    acidity: string,
    tannin: string,       // For red wines
    body: string,
    finish: string,
    additionalNotes: string
  },
  notes: string,          // Legacy notes field
  createdAt: string,      // ISO timestamp
  updatedAt: string       // ISO timestamp
}
```

### Photo Handling & Optimization

Photos are automatically compressed and stored as Base64-encoded data URLs in IndexedDB:

**Image Processing Pipeline**:
1. User selects image → `handlePhotoSelect()` in [src/js/app.js](src/js/app.js)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/773hinako) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
