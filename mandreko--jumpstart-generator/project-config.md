---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MTG Jumpstart Card Generator that converts WOTC text file deck lists into printable cards through a two-phase pipeline:
1. Parse WOTC text files into intermediate JSON format
2. Query Scryfall API, process images, and generate Card Conjurer JSON files

The end result: ready-to-print custom Jumpstart packs for makeplayingcards.com (MPC).

## Common Commands

### Build Pipeline
```bash
npm run rebuild    # Full rebuild: clean + build + render (takes ~10-15 min)
npm run build      # Convert + generate (skips clean and render, ~5-10 min)
npm run render     # Phase 3: Render card backs via Playwright (~4-5 min parallel)
npm run convert    # Phase 1: Parse WOTC text files to JSON
npm run generate   # Phase 2: Generate Card Conjurer files + front images
npm run clean      # Delete all generated output files
```

**Note:** `rebuild` includes automated card back rendering via Playwright. Use `build` for faster iteration when you don't need back images.

**Parallelization:** The render phase runs all sets in parallel by default. Limit with `RENDER_PARALLEL=2 npm run render` if needed.

### Single Set Processing
```bash
# Convert specific set's text files
node convert-wotc-txt-to-json.js "Avatar"

# Generate Card Conjurer files for specific set
node generate-card-conjurer-json.js TLA

# Available set codes: TLA, DMU, J25, J22, LTR, MOM, ONE, BRO
```

## Architecture & Data Flow

```
INPUT: txt-from-wotc/{set-name}/*.txt
  ↓
convert-wotc-txt-to-json.js (Phase 1)
  ├─ Regex parsing: ^(\d+)\s+(.+)$
  ├─ Groups cards by pack (filename)
  └─ Deduplicates and sums quantities
  ↓
OUTPUT: output/json-decklists/{set-name}-output.json
  ↓
generate-card-conjurer-json.js (Phase 2)
  ├─ Queries Scryfall API (rate-limited 100ms)
  ├─ Downloads card images (672×936px)
  ├─ Processes with ImageMagick:
  │   • Overlays 28px black on borders
  │   • Adds 33px bleed border (final: 738×1002px)
  ├─ Generates Card Conjurer JSON from template
  └─ Calculates watermark positioning
  ↓
OUTPUT (3 types):
  ├─ output/cardconjurer-json-files/*.json (individual cards)
  ├─ output/cardconjurer-import-files/*.cardconjurer (bulk import)
  └─ output/front-images/*.jpg (front images, ~125MB total)
  ↓
render-card-backs.js (Phase 3)
  ├─ Launches headless Chromium via Playwright
  ├─ Uploads .cardconjurer file to CardConjurer.app
  ├─ Iterates through saved cards dropdown
  ├─ Resets watermark and downloads each card
  └─ Saves rendered images with matching filenames
  ↓
OUTPUT: output/back-images/*.jpg (back images, ~125MB total)
```

## Key Configuration: sets.json

Master configuration for all 8 supported sets. Each entry contains:
- `background-watermark`: SVG URL from github.com/pappnu/mtg-vectors
- `lower-watermark`: Set symbol SVG URL
- `intermediate-json`: Output filename for Phase 1

Set codes: TLA, DMU, J25, J22, LTR, MOM, ONE, BRO

## Core Processing (generate-card-conjurer-json.js)

This 1,044-line file is the processing engine. Key components:

### Caching System
- **ScryfallCache**: Two maps prevent redundant API calls
  - `cardDataCache`: card name → {name, mana_cost, type_line}
  - `packCardCache`: "{set}:{packName}" → {themeColor, collectorNumber, imageUri}

### Rate Limiting
- **RateLimiter class**: Enforces 100ms delay between Scryfall requests
- Required by Scryfall API terms

### Image Processing Pipeline
1. Download from Scryfall (672×936px)
2. `overlayBlackOnOriginalBorder()`: 28px black on edges via ImageMagick
3. `addBlackBorder()`: Add 33px bleed border (final: 738×1002px)

### Template System
- **CARD_TEMPLATE**: Embedded Card Conjurer format (lines 34-113)
- Color frames mapped from COLOR_TO_FRAME_MAP: {W/U/B/R/G/C/multicolor}
- Deep clone pattern: `JSON.parse(JSON.stringify(template))`

### Key Functions
| Function | Purpose |
|----------|---------|
| `queryCardData()` | Scryfall API query with caching |
| `queryPackCard()` | Find theme color from pack face card |
| `calculateWatermarkPosition()` | Fit watermark to bounds with zoom |
| `generateRulesText()` | Format card list by type (Creature, Instant, etc.) |
| `generatePackJSON()` | Create final Card Conjurer JSON |

## File Naming Conventions

**Output files follow this pattern:**
- JSON cards: `{SET}-{COLLECTOR_NUMBER}-{PACK_NAME}.json`
- Front images: `{SET}-front-{COLLECTOR_NUMBER}-{PACK_NAME}.jpg`
- Back images: `{SET}-back-{COLLECTOR_NUMBER}-{PACK_NAME}.jpg`
- Import file: `{SET}-saved-cards.cardconjurer`

**Examples:**
- `J22-front-0001-Blink-1.jpg` (front image)
- `J22-back-0001-Blink-1.jpg` (back image)

**Collector numbers:** `F 0001` format (F = Jumpstart Face card, 4 digits)

## External Dependencies

### Required Tools
- **Node.js**: v14+ (specified in package.json engines)
- **ImageMagick**: CLI tool for image processing
  - Commands used: `magick identify`, `magick ... -fill ... -draw ...`
- **Playwright**: Browser automation for rendering card backs
  - Installed via npm as devDependency
  - Uses headless Chromium

### APIs
- **Scryfall API** (api.scryfall.com)
  - `/cards/named?exact={name}` - Get card data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mandreko/jumpstart-generator](https://github.com/mandreko/jumpstart-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
