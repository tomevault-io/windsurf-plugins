---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FF14.tw is a multi-tool website for Final Fantasy XIV players in Taiwan, providing various utilities like character card generators, dungeon database, and game calculators. The project uses a vanilla web stack with a modular architecture where each tool is self-contained within the `tools/` directory.

## Architecture

- **Static Website**: Pure HTML/CSS/JavaScript with no build tools, bundlers, or frameworks
- **Multi-Tool Structure**: Each tool is completely self-contained in its own directory under `tools/`
- **Shared Resources**: Common utilities in `assets/` (CSS variables, utility functions, constants)
- **Data Management**: JSON files in `/data/` directory for large datasets (dungeons, treasure maps, translations)
- **Language**: Traditional Chinese (zh-Hant) - all UI text and content
- **I18n Support**: Multi-language support (zh/en/ja) via I18nManager
- **Deployment**: GitHub Pages with custom domain (ff14.tw via CNAME)

## Project Statistics

- **HTML Files**: 24 total — 19 across 12 tool directories (11 single-page tools + `guide/` with 8 pages) + 4 main pages (`index.html`, `about.html`, `changelog.html`, `copyright.html`) + 1 API test harness (`api/test.html`)
- **JavaScript Files**: 76 total — 36 tool scripts + 4 shared utilities (`assets/js/`) + 17 i18n (manager + translations) + 3 layout components (`assets/js/components/`) + 15 test files (`tests/*.test.js`) + 1 Cloudflare Worker (`api/treasure-room-worker.js`); plus 3 Node build scripts (`scripts/*.mjs`, not shipped to the site)
- **CSS Files**: 23 (shared + components + tool-specific)
- **JSON Data Files**: 8 (total ~25,600 lines); `/data/` also has one non-JSON `ff14-gp.csv`
- **Total Dungeons**: 803 entries (data file's own `metadata.totalDungeons` field still says 804 — pre-existing inconsistency inside `dungeons.json` itself, not a doc bug)
- **Total Treasure Map Coordinates**: 227 entries (G8/G10/G12/G14/G15/G17/G18)

## Development Commands

This is a static website with **no build process**. Files can be edited directly and changes are reflected immediately.

**Local Development:**
```bash
# Recommended: Use local server (required for tools with JSON data)
python3 -m http.server
# Access at: http://localhost:8000

# Alternative servers:
npx serve .
# or
php -S localhost:8000
```

**CORS Requirements:**
Tools that fetch JSON data require a local server:
- 副本資料庫 (`dungeon-database/`) - loads `/data/dungeons.json`
- 寶圖搜尋器 (`treasure-map-finder/`) - loads `/data/aetherytes.json`, `/data/treasure-maps.json` and `/data/zones.json`
- Lodestone 角色查詢 (`lodestone-lookup/`) - uses logstone API
- 特殊採集時間管理器 (`timed-gathering/`) - loads `/data/timed-gathering.json`
- 巨集轉換器 (`macro-converter/`) - loads `/data/macro-mappings.json`
- 攻略資料 (`guide/`) - 僅陸行鳥毛色頁面 (`chocobo.html`) loads `/data/chocobo-colors.json`，其餘 7 個頁面不需要伺服器

Tools that work without server (can open HTML directly):
- 仙人微彩計算機
- Wondrous Tails 預測器
- 角色卡產生器
- Faux Hollows Foxes 計算機
- 檢舉模板產生器
- 天氣預報
- 攻略資料（陸行鳥毛色頁面除外）

**Testing:** Node 24；先執行 `npm --prefix api ci` 安裝 Miniflare/workerd，再執行 `node --test`（Node 內建 test runner，自動執行 `tests/*.test.js`）。`tests/design-system.test.js` 守住設計系統規則（token 完整性、對比度、token-clean 檔案清單）；`tests/pages.test.js` 守住每一頁的載入順序與字型；`tests/timed-gathering-eorzea-time.test.js` 守住艾歐澤亞時間換算在不同時區下的一致性；`tests/scripts.test.js` 守住 `assets/`、`tools/` 底下的 JS 不得使用 `innerHTML`；`tests/docs.test.js` 守住 CLAUDE.md／README 的副本數、寶圖座標數與 JSON 資料檔案數這幾個關鍵統計數字不會與 `/data` 底下的實際資料脫節；`tests/modal-manager-stack.test.js` 用最小 DOM 替身（不需 jsdom）守住 `ModalManager` 的共用堆疊行為：只有最上層回應 Escape／焦點陷阱、關閉下層時由上而下連鎖收合、焦點依序回捲。新增行為測試涵蓋寶圖前後端契約、SQLite Durable Object 並行與權限、備份還原、通知降級、時間解析、陸行鳥全部色對、宗長候選、Lodestone 請求競爭、i18n 儲存與天氣網址狀態。Worker 測試需要允許 localhost 連接埠；`.github/workflows/test.yml` 在 push／PR 自動執行全套測試與兩個 Worker 環境的 dry-run。每次 commit 前執行。網站主體沒有 package.json、bundler 或 linter（`api/` 的 Cloudflare Worker 子專案另有自己的 `package.json`／`wrangler`，與網站建置無關）。

## Core Patterns

### Tool JavaScript Architecture
Each tool uses a consistent class-based pattern:

```javascript
class ToolCalculator {
    // Constants definition at class level
    static CONSTANTS = {
        DEBOUNCE_DELAY: 300,
        CSS_CLASSES: {
            ACTIVE: 'active',
            FOCUSED: 'focused'
        }
    };

    constructor() {
        this.state = {};
        this.elements = {
            grid: document.getElementById('tool-grid'),
            result: document.getElementById('result-display')
        };
        this.initializeEvents();
    }
    
    initializeEvents() {
        // Use named methods for removable event handlers
        this.handleClick = (e) => { /* handler logic */ };
        this.elements.grid.addEventListener('click', this.handleClick);
    }
}
```

### Multi-Select Tag Filtering Pattern
Modern tools implement multi-select filtering with tag buttons:

```javascript
// State management with Sets for O(1) lookup performance
this.selectedTypes = new Set();
this.selectedExpansions = new Set();

// Toggle method pattern
toggleTypeTag(tagElement) {
    const type = tagElement.dataset.type;
    if (this.selectedTypes.has(type)) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hydai/ff14.tw](https://github.com/hydai/ff14.tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
