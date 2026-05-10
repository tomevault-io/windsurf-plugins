---
trigger: always_on
description: **Calcolo Diluizioni** (Dilution Calculator) is a single-page web application for calculating product dilution ratios, primarily targeting car care and detailing products. The app is written entirely in Italian.
---

# CLAUDE.md - AI Assistant Guidelines for Calcolo Diluizioni

## Project Overview

**Calcolo Diluizioni** (Dilution Calculator) is a single-page web application for calculating product dilution ratios, primarily targeting car care and detailing products. The app is written entirely in Italian.

## Quick Start

The frontend (`diluizioni.html`) talks to a PHP backend (`api.php`) that reads
and writes `products.json`, so a PHP-capable server is required — a plain
static server is no longer enough.

```bash
# From the repo root
php -S localhost:8000
# Then open http://localhost:8000/diluizioni.html
```

For the admin area to work in local dev, create a `config.php` with a bcrypt
hash (the file is git-ignored):

```bash
php -r "echo \"<?php\nconst ADMIN_PASSWORD_HASH = '\" . password_hash('admin', PASSWORD_BCRYPT) . \"';\n\";" > config.php
```

## Project Structure

```
calcolo-diluizioni/
├── CLAUDE.md            # This file - AI assistant guidelines
├── README.md            # Project description (Italian)
├── diluizioni.html      # Frontend SPA (HTML + CSS + JS in one file)
├── api.php              # Backend REST-ish API (products CRUD + admin auth)
├── products.json        # Persistent product catalog (written by api.php)
├── config.php           # [git-ignored] admin password hash
├── setup.php            # [git-ignored] one-shot admin password setup helper
└── lxc/                 # LXC deployment documentation
    ├── README.md        # Production snapshot + redeploy runbook
    └── nginx-site.conf  # Versioned copy of the production Nginx site
```

### File Organization (diluizioni.html)

| Section | Lines | Description |
|---------|-------|-------------|
| HTML Head & Meta | 1-6 | Document declaration, UTF-8, viewport |
| CSS Styles | 7-320 | Complete inline stylesheet |
| HTML Structure | 321-445 | Tab-based UI with three sections |
| JavaScript | 447-781 | Application logic and data |
| Closing Tags | 782-783 | `</body>` and `</html>` |

## Architecture

### Technology Stack

- **Frontend**: HTML5 + CSS3 + Vanilla ES6+ in a single file (`diluizioni.html`)
- **Backend**: PHP 8.x (`api.php`) — stateless except for a PHP session used for admin auth
- **Persistence**: flat JSON file (`products.json`) written by `api.php`
- **Web server**: Nginx + PHP-FPM (production, LXC container)
- **No build process**: the frontend is served as-is, no bundler, no transpiler

### Key Design Decisions

1. **Frontend in one file**: all HTML, CSS, and JS in `diluizioni.html` for simplicity
2. **Minimal backend**: `api.php` is a single PHP file with a handful of REST-ish endpoints; no framework
3. **No database**: products live in `products.json` on disk, read/written by `api.php`. Good enough for the scale of the app.
4. **Mobile-first**: Container max-width 480px with responsive adjustments
5. **State-driven UI**: Simple state variables (`selectedVolume`, `selectedRatio`, etc.)

## Core Features

### 1. Calcolatore (Calculator Tab)
- Main dilution calculator using ratio format 1:X
- Preset buttons for common volumes and ratios
- Auto-calculation when presets selected

### 2. Rabbocco (Refill Tab)
- Calculates how to refill bottles with existing diluted product
- Validates feasibility and shows appropriate error messages
- Tracks existing concentrate in bottle

### 3. Riferimento Prodotti (Products Tab)
- Reference guide for the supported cleaning products
- Each product has multiple usage scenarios with recommended ratios
- Clicking a use auto-sets the ratio and switches to Calculator tab
- The list is loaded at startup via `GET /api.php` (reads `products.json`)

### 4. Area Admin (password protected)
- Login via `POST /api.php?action=login` (bcrypt password check in `config.php`)
- Session-based auth (PHP `$_SESSION['admin']`)
- CRUD operations on products: add / update / delete, persisted in `products.json`
- UI exposed inside `diluizioni.html` when the user is authenticated

## Code Conventions

### Naming

- **Functions**: camelCase (`calculateRefill`, `renderVolumePresets`)
- **Variables**: camelCase (`selectedVolume`, `volumePresets`)
- **CSS Classes**: kebab-case (`tab-content`, `btn-primary`, `result-row`)
- **IDs**: kebab-case (`calc-result`, `product-select`)

### JavaScript Patterns

```javascript
// State management - simple variables
let selectedVolume = '';
let selectedRatio = '';

// Render functions - return HTML strings via template literals
function renderVolumePresets() {
  container.innerHTML = volumePresets.map(v =>
    `<button class="preset-btn" onclick="selectVolume(${v})">${formatVolume(v)}</button>`
  ).join('');
}

// Event handlers - inline onclick for presets, addEventListener for inputs
document.getElementById('volume').addEventListener('input', function() { ... });
```

### CSS Color System (Tailwind-inspired)

- Primary Blue: `#2563eb`
- Cyan: `#0891b2`
- Green: `#16a34a`
- Red: `#dc2626`
- Gray scale: `#f9fafb` to `#1f2937`

## Key Calculation Formulas

### Basic Dilution (1:X ratio)
```javascript
// For ratio 1:X, total parts = 1 + X
totalParts = 1 + ratio;
product = volume / totalParts;
water = product * ratio;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexpani/diluition-calc](https://github.com/alexpani/diluition-calc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
