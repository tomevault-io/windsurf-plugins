---
trigger: always_on
description: A Flask-based web gallery application where users can upload artwork images that display on a tiled wall. Each tile can be clicked to view the full-size image with metadata displayed in an animated overlay ribbon.
---

# The Last Gallery - Project Summary

## Overview
A Flask-based web gallery application where users can upload artwork images that display on a tiled wall. Each tile can be clicked to view the full-size image with metadata displayed in an animated overlay ribbon.

## Tech Stack
- **Backend**: Python/Flask
- **Database**: SQLite (single source of truth) with versioned migrations
- **Frontend**: Vanilla JavaScript (modular), CSS
- **Image Processing**: Cropper.js (client-side cropping), Pillow (server-side optimization)
- **Payments**: Stripe Checkout (tile upgrades), webhook-only fulfillment
- **Email**: Resend API (edit code delivery), `python-dotenv` for env config

## How to Run
```bash
pip install -r requirements.txt
python app.py
```
- Local: http://127.0.0.1:5000
- Network: http://192.168.1.191:5000 (for mobile testing)

## Key Files

### Backend
| File | Purpose |
|------|---------|
| `app.py` | Flask application, all API endpoints |
| `db.py` | Database connection, schema initialization, versioned migrations |
| `data/gallery.db` | SQLite database (schema v22) |
| `select_cotm.py` | Monthly COTM winner selection (runs via systemd timer) |
| `grid utilities/repair_tiles.py` | Sync tiles table with SVG after grid extension |
| `cleanup_expired.py` | Remove artwork past its 24-hour payment deadline (runs via systemd timer) |

### Frontend
| File | Purpose |
|------|---------|
| `static/js/main.js` | Core gallery rendering, popup overlay, wall state management |
| `static/js/admin.js` | Admin modal, action handlers (clear/move/undo/shuffle), countdown admin controls |
| `static/js/countdown.js` | Countdown timer bar module (fetch state, tick, show/hide) |
| `static/js/unlock_modal.js` | Upgrade modal: 3-step purchase flow (identify → select artwork → choose tier) |
| `static/js/upload_modal.js` | Image upload flow with cropping, metadata entry, and COTM opt-in |
| `static/js/cotm.js` | Creator of the Month popup, edit form, auto-show |
| `static/js/deadline_banner.js` | 24-hour unlock deadline banner shown after 2nd+ uploads |
| `templates/index.html` | Main HTML template |
| `static/css/styles.css` | All styling including popup animations |
| `static/grid_full.svg` | SVG defining tile positions and sizes |

## Environment Variables
| Variable | Default | Purpose |
|----------|---------|---------|
| `TLG_ADMIN_PIN` | *(required)* | 5-digit admin PIN for admin endpoints |
| `TLG_BASE_URL` | `https://thelastgallery.com` | Base URL used in email links |
| `RESEND_API_KEY` | *(none)* | Resend API key for sending edit code emails |
| `STRIPE_SECRET_KEY` | *(none)* | Stripe secret key for payment processing |
| `STRIPE_WEBHOOK_SECRET` | *(none)* | Stripe webhook signing secret |

## Admin PIN
- 5-digit PIN, set via `TLG_ADMIN_PIN` in `.env` (required, no hardcoded default)
- Validated server-side only; never exposed to client-side JavaScript
- Rate limiting: 5 failed attempts per IP → 15-minute lockout (HTTP 429)
- PIN stored in IIFE closure scope after successful server validation, persists until page refresh

## JavaScript Architecture

### Global Exposure (from main.js)
```javascript
window.DEBUG          // Debug mode flag
window.ADMIN_DEBUG    // Admin debug footer flag
window.SEL            // DOM selector constants
window.API            // API endpoint constants
window.PAGE_MODE      // "edit" | "creator-of-the-month" | "purchase_success" | "upgrade" | "art" | ""
window.refreshWallFromServer()    // Refresh wall from database
window.refreshAdminOverlays()     // Refresh admin UI (from admin.js)
window.isAdminActive()            // Check admin session (from admin.js)
window.getAdminPin()              // Get admin PIN for cross-module requests (from admin.js)
window.initZoom()                 // Initialize pinch-to-zoom
window.resetZoom()                // Reset zoom to 1.0x
window.highlightNewTile(tileId)   // Scroll to tile + sheen animation
window.showShareToast()           // Show "Link copied" toast notification
window.refreshCountdown()         // Re-fetch countdown state (from countdown.js)
window.openUnlockModal(assetId, tileId)  // Open unlock modal (from unlock_modal.js)
window.openFloorUpgrade(artwork, editCode)  // Direct floor upgrade (from unlock_modal.js)
window.closeUnlockModal()         // Close unlock modal (from unlock_modal.js)
window.isUnlockModalOpen()        // Check if unlock modal is open (from unlock_modal.js)
window.registerDismissible(overlayId, closeBtnId, hashName)  // Register overlay for unified dismiss
window.closeAboutExhibits(silent)  // Close About Exhibits overlay (from exhibit.js)
window.showDeadlineBanner(opts)    // Show 24-hour deadline banner (from deadline_banner.js)
window.dismissDeadlineBanner()     // Dismiss deadline banner (from deadline_banner.js)
```

### Dismissible Overlay Registry (main.js)
- `registerDismissible()` registers overlays for unified close: close button, tap-anywhere, Escape, back button
- Returns `{open(silent), close(silent), overlay, hashName, isOpen}`
- `silent` param skips hash push/pop (for sub-overlays)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/989Daren) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
