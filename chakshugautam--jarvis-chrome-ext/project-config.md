---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JARVIS is a Chrome extension (Manifest V3) that applies intelligent performance optimizations to specific websites using monkey-patching techniques. Currently supports:
- **Hasgeek.com**: Performance optimizations (lazy loading, redirect fixes, image optimization)
- **The Sun (thesun.co.uk)**: Ad blocking and content cleanup

## Development Commands

### Installing the Extension

1. Open Chrome: `chrome://extensions/`
2. Enable "Developer mode" (top-right toggle)
3. Click "Load unpacked"
4. Select directory: `/Users/__chaks__/jarvis-extension`

### Testing

**Test on Hasgeek:**
```
Visit: https://hasgeek.com/rootconf/
Open DevTools → Console
Filter logs by: [JARVIS]
```

**Test on The Sun:**
```
Visit: https://www.thesun.co.uk/
Check console for blocked ads/scripts
```

### Debugging

- Console logs are prefixed with `[JARVIS]`
- Stats are tracked per-session (reset on reload)
- Check extension errors: `chrome://extensions/` → "Errors" button

## Architecture

### Core Components

**manifest.json**
- Defines permissions, content scripts, and background service worker
- Content scripts run at `document_start` for early DOM interception
- Uses declarativeNetRequest for network-level blocking

**background.js** (Service Worker)
- Manages extension lifecycle and settings
- Controls declarativeNetRequest rule enabling/disabling
- Handles message passing between popup and content scripts
- Temporarily unblocks Razorpay when payment buttons clicked

**popup.html + popup.js**
- UI for toggling features and viewing stats
- Site-specific panels (Hasgeek vs The Sun vs unsupported)
- Real-time stats updates via message passing

**Content Scripts**
- `scripts/hasgeek-patches.js`: Hasgeek optimizations
- `scripts/thesun-patches.js`: The Sun ad blocking

**rules.json**
- Declarative network request rules for blocking scripts
- Blocks Razorpay, ad networks, tracking scripts

### Monkey-Patching Techniques

The extension uses aggressive runtime patching to intercept browser behavior:

**1. createElement Override**
```javascript
document.createElement = function(tagName) {
  // Intercept script tags to block Razorpay
}
```

**2. Property Descriptor Patching**
```javascript
Object.defineProperty(HTMLImageElement.prototype, 'src', {
  set: function(value) {
    // Rewrite image URLs to bypass redirects
  }
})
```

**3. MutationObserver**
- Watches for dynamically added scripts/images/ads
- Removes blocked elements before they execute/load

**4. fetch() Override**
```javascript
window.fetch = function(...args) {
  // Block OpenStreetMap tiles until venue visible
}
```

**5. IntersectionObserver**
- Detects when venue section enters viewport
- Triggers map loading only when needed

## Hasgeek Optimizations

### Patch 1: Lazy Load Razorpay (~450KB savings)
- Blocks all Razorpay scripts via declarativeNetRequest rules
- Overrides `createElement` to catch dynamic script injection
- When payment button clicked: unblocks Razorpay, injects checkout script, re-triggers click
- Background worker re-enables blocking after 60 seconds

### Patch 2: Fix Image Redirects (~150ms per image)
- Detects `images.hasgeek.com/embed/file/{hash}` URLs
- Extracts hash and size, rewrites to direct S3 URL: `imgee.s3.amazonaws.com/imgee/{hash}_w{size}_h{size}.jpeg`
- Patches both `img.src` setter and `setAttribute`

### Patch 3: Lazy Load Images
- Adds `loading="lazy"` attribute to images (skips first 3)
- MutationObserver catches dynamically added images

### Patch 4: Lazy Load Maps (~200KB savings)
- Overrides `fetch()` to block `openstreetmap.org` tiles
- IntersectionObserver detects when venue section visible
- Restores original `fetch()` when map needed

## The Sun Optimizations

### Patch 1: Block Ad Scripts
- MutationObserver removes script tags from 20+ ad/tracking domains
- Includes Google Ads, DoubleClick, Facebook, TikTok, analytics

### Patch 2: Remove Ad Containers
- Removes iframes and divs matching ad patterns
- Runs on load + every 3 seconds (catches late-loading ads)
- Checks: ad-related IDs/classes, Google query IDs, large empty containers

### Patch 3: Clean Reading Mode
- Removes recommendation sections ("READ MORE", "MOST READ", etc.)
- Removes sidebars, footers, widget sections
- Removes injected promotional boxes (`.article-boxout`)
- Removes newsletter signup forms (`.inline-module--newsletter`)
- Removes promotional paragraphs (casino/betting affiliate links)
- Preserves main article content and navigation
- Specific selectors: `.digital-personalisation`, `.single__sidebar`, etc.

### Patch 4: Remove Images (Optional)
- Removes all `<img>` and `<picture>` elements from article content
- Scoped to `.article__content` to avoid breaking layout
- MutationObserver watches for dynamically added images

### Patch 5: Remove Videos (Optional)
- Removes `<video>` elements and Brightcove players
- Removes video containers by class name matching
- MutationObserver watches for dynamically added videos

## Hasgeek Additional Patches

### Patch 5: Remove Images (Optional)
- Removes all `<img>` and `<picture>` elements from entire page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChakshuGautam/jarvis-chrome-ext](https://github.com/ChakshuGautam/jarvis-chrome-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
