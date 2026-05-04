---
trigger: always_on
description: **Hong Kong Open Map** is a free, open-source map component library for Hong Kong applications. It uses official vector tiles from the Hong Kong Lands Department (地政總署) and provides a perfect alternative to Google Maps for Hong Kong-focused applications.
---

# GitHub Copilot Instructions | GitHub Copilot 指引

## Project Overview | 項目概述

**Hong Kong Open Map** is a free, open-source map component library for Hong Kong applications. It uses official vector tiles from the Hong Kong Lands Department (地政總署) and provides a perfect alternative to Google Maps for Hong Kong-focused applications.

**香港開放地圖** 是一個免費、開源的香港地圖組件庫。使用香港地政總署官方矢量圖磚，為香港應用程式提供 Google Maps 的完美替代方案。

---

## Tech Stack | 技術棧

| Technology | Purpose | 用途 |
|------------|---------|------|
| React 18+ | Primary component library | 主要組件庫 |
| TypeScript | Type safety and DX | 類型安全與開發體驗 |
| MapLibre GL JS | Map rendering engine (free & open-source) | 地圖渲染引擎（免費開源）|
| Vite | Build tool | 構建工具 |
| Vitest | Testing framework | 測試框架 |
| Rollup | Library bundling | 庫打包 |

---

## Distribution Formats | 發佈格式

This package supports multiple usage methods:

1. **React Component** (`hong-kong-open-map`)
   - Primary npm package for React applications
   - React 應用程式的主要 npm 套件

2. **Vanilla JavaScript** (`hong-kong-open-map/vanilla`)
   - Standalone JS for non-React projects
   - 適用於非 React 項目的獨立 JS

3. **iFrame Embed** (`https://wangwailok.com/hong-kong-open-map/embed`)
   - Public URL for easy embedding
   - 公開 URL 方便嵌入

---

## Core API Reference | 核心 API 參考

### Map Tile URLs | 地圖圖磚網址

```typescript
// Base vector tile URL (WGS84 projection for MapLibre)
const BASEMAP_URL = "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/vt/basemap/WGS84";

// Label layers (Chinese/English)
const LABEL_URL_ZH = "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/vt/label/hk/tc/WGS84";
const LABEL_URL_EN = "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/vt/label/hk/en/WGS84";

// Style JSON URL (for MapLibre)
const STYLE_URL = "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/vt/basemap/WGS84/style.json";
const STYLE_URL_DARK = "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/vt/basemap/WGS84/style_dark.json";
```

### Spatial Reference | 空間參考

```typescript
// WGS84 (EPSG:4326) - Standard for MapLibre GL JS
const WGS84_EPSG = 4326;

// Default center point (Victoria Harbour area)
const DEFAULT_CENTER: [number, number] = [114.1694, 22.3193]; // [longitude, latitude]

// Default bounds for Hong Kong
const HK_BOUNDS: [[number, number], [number, number]] = [
  [113.8259, 22.1533],  // Southwest
  [114.4069, 22.5620]   // Northeast
];
```

### Zoom Constraints | 縮放限制

```typescript
const ZOOM_CONSTRAINTS = {
  minZoom: 8,   // Hong Kong overview
  maxZoom: 19,  // Street level detail
  defaultZoom: 10
};
```

---

## Component Props Interface | 組件屬性介面

```typescript
interface HKMapProps {
  // Map configuration
  center?: [number, number];     // [longitude, latitude] in WGS84
  zoom?: number;                 // Initial zoom level (8-19)
  theme?: 'light' | 'dark' | 'auto';
  language?: 'zh' | 'en' | 'auto';
  
  // Styling
  className?: string;
  style?: React.CSSProperties;
  width?: string | number;
  height?: string | number;
  
  // Markers
  markers?: HKMapMarker[];
  
  // Events
  onLoad?: (map: MapInstance) => void;
  onClick?: (event: MapClickEvent) => void;
  onMarkerClick?: (marker: HKMapMarker) => void;
  
  // Accessibility
  ariaLabel?: string;
  tabIndex?: number;
}

interface HKMapMarker {
  id: string;
  position: [number, number];    // [longitude, latitude] in WGS84
  title?: string;
  popup?: string | React.ReactNode;
  icon?: string | MarkerIcon;
}
```

---

## File Structure | 檔案結構

```
hong-kong-open-map/
├── src/
│   ├── index.ts              # Main entry point
│   ├── react/                # React components
│   │   ├── HKMap.tsx         # Main map component
│   │   ├── HKMarker.tsx      # Marker component
│   │   ├── HKPopup.tsx       # Popup component
│   │   └── hooks/            # React hooks
│   │       ├── useMap.ts
│   │       └── useMarkers.ts
│   ├── vanilla/              # Vanilla JS implementation
│   │   ├── index.ts
│   │   ├── HKMap.ts
│   │   └── utils.ts
│   ├── core/                 # Shared core logic
│   │   ├── config.ts         # Map configuration
│   │   ├── tiles.ts          # Tile layer management
│   │   ├── themes.ts         # Theme definitions
│   │   ├── projection.ts     # Coordinate conversion
│   │   └── types.ts          # TypeScript types
│   ├── styles/               # CSS styles
│   │   ├── map.css
│   │   └── themes/
│   └── embed/                # iFrame embed page
│       └── index.html
├── dist/                     # Built files
├── docs/                     # Documentation
├── examples/                 # Usage examples
└── tests/                    # Test files
```

---

## Coding Standards | 編碼標準

### General Rules | 一般規則

1. **Comments in English only** - All code comments should be in English
2. **Documentation bilingual** - All user-facing docs in Chinese and English
3. **TypeScript strict mode** - Enable all strict type checking
4. **Named exports preferred** - Use named exports over default exports
5. **Functional components** - Use React functional components with hooks

### Naming Conventions | 命名慣例

```typescript
// Components: PascalCase with HK prefix
HKMap, HKMarker, HKPopup


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangwailok/hong-kong-open-map](https://github.com/wangwailok/hong-kong-open-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
