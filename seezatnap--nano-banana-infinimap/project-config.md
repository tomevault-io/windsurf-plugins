---
trigger: always_on
description: A local-first, generative slippy map application that creates tiles on-demand using AI generation. Built with Next.js, TypeScript, and Leaflet, it features neighbor-aware tile generation for seamless edges.
---

# Infinimap - Generative Slippy Map

## Project Overview
A local-first, generative slippy map application that creates tiles on-demand using AI generation. Built with Next.js, TypeScript, and Leaflet, it features neighbor-aware tile generation for seamless edges.

## Architecture

### Tech Stack
- **Next.js 15** with App Router
- **TypeScript** for type safety
- **Tailwind CSS** for styling (v4 with @tailwindcss/postcss)
- **Leaflet** for map rendering (Simple CRS)
- **Sharp** for image processing
- **Zod** for validation
- **File-based storage** (no external databases)

### Key Design Decisions
1. **Local-first**: Everything runs locally without external services
2. **File-based adapters**: DB, Queue, and Storage use filesystem with clean interfaces for future cloud migration
3. **Neighbor-aware generation**: Tiles know about adjacent tiles for edge continuity
4. **Content hashing**: Merkle tree structure for efficient cache invalidation
5. **URL state management**: Map position tracked in query parameters for sharing

## Project Structure

```
infinimap/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── tiles/[z]/[x]/[y]/    # Serve tiles
│   │   ├── claim/[z]/[x]/[y]/    # Generate tiles
│   │   ├── invalidate/[z]/[x]/[y]/ # Regenerate tiles
│   │   └── meta/[z]/[x]/[y]/     # Tile metadata
│   └── map/               # Map viewer page
├── components/
│   └── MapClient.tsx      # Main map component with URL state
├── lib/
│   ├── adapters/          # Swappable implementations
│   │   ├── db.file.ts     # File-based tile metadata
│   │   ├── queue.file.ts  # In-process job queue
│   │   └── lock.file.ts   # File-based locks
│   ├── generator.ts       # Tile generation logic (STUB)
│   ├── hashing.ts         # Content hashing & Merkle tree
│   ├── storage.ts         # Tile image storage
│   └── coords.ts          # Coordinate math
├── public/
│   ├── default-tile.webp  # Placeholder for empty tiles
│   └── style-control/     # Style configuration
│       └── config.json    # Generation parameters
└── .tiles/                # Generated tile images (gitignored)
    .meta/                 # Tile metadata (gitignored)
    .locks/                # Lock files (gitignored)
    .queue/                # Queue state (gitignored)
```

## Key Features

### Map Navigation
- **URL State Management**: Position tracked via query params (?z=4&lat=-128&lng=128)
- **Shareable Links**: Bookmark or share specific locations
- **Position Restoration**: Refreshing maintains view position
- **Real-time position display**: Shows current coordinates in UI

### Tile Generation
- **Click to Generate**: At max zoom (level 8), click any tile to generate
- **Prompt System**: Describe areas with text prompts
- **Neighbor Awareness**: Generator receives adjacent tiles for continuity
- **Style Control**: Centralized style configuration in `/public/style-control/config.json`
- **Generation Feedback**: Popup notifications and auto-refresh on completion

### Performance
- **Lazy Generation**: Tiles only generated when requested
- **Immutable Caching**: Generated tiles cached forever via ETags
- **Debounced Updates**: Map position updates throttled to 300ms
- **Polling System**: Auto-refresh tiles when generation completes (30-second timeout)

## API Endpoints

### GET /api/tiles/{z}/{x}/{y}
Returns tile image (WebP). Falls back to default-tile.webp if not generated.
- Uses async params (Next.js 15 requirement)
- Returns immutable cache headers with ETag

### POST /api/claim/{z}/{x}/{y}
```json
{ "prompt": "isomorphic video game layout" }
```
Initiates tile generation at max zoom level only.

### POST /api/invalidate/{z}/{x}/{y}
Regenerates existing tile with incremented version.

### GET /api/meta/{z}/{x}/{y}
Returns tile metadata:
```json
{
  "status": "EMPTY" | "PENDING" | "READY",
  "hash": "abc123...",
  "updatedAt": "2024-01-01T00:00:00Z"
}
```

## Configuration

### Environment Variables (.env.local)
```bash
ZMAX="8"                    # Maximum zoom level
TILE_SIZE="256"            # Tile dimensions in pixels
DEFAULT_TILE_PATH="./public/default-tile.webp"
STYLE_PATH="./public/style-control/config.json"
STYLE_REF="./public/style-control/ref.png"  # Optional style reference
NEXT_PUBLIC_ZMAX="8"       # Client-side max zoom
```

### Style Configuration (public/style-control/config.json)
```json
{
  "name": "isomorphic-v1",
  "palette": {
    "deep": "#143C82",
    "shallow": "#1E5AA0",
    "beach": "#F0E6B4",
    "grass": "#328C3C",
    "hills": "#5B503C",
    "snow": "#E6E6E6"
  },
  "model": { 
    "sampler": "dpmpp_2m", 
    "steps": 25, 
    "cfg": 5.5 
  }
}
```

## Development

### Prerequisites
- Node.js 18+
- Yarn package manager

### Running Locally
```bash
yarn install    # Install dependencies
yarn dev        # Start development server
yarn build      # Build for production
yarn start      # Run production build
```

### Testing Tile Generation
1. Navigate to http://localhost:3000/map
2. Zoom to maximum level (8)
3. Enter a prompt in the input field
4. Click any tile to generate it
5. Watch the popup for generation status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seezatnap/nano-banana-infinimap](https://github.com/seezatnap/nano-banana-infinimap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
