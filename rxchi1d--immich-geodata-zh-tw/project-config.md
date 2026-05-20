---
trigger: always_on
description: 此檔案提供 Claude Code (claude.ai/code) 在此儲存庫中工作時的指引。
---

# CLAUDE.md

此檔案提供 Claude Code (claude.ai/code) 在此儲存庫中工作時的指引。

## 語言規則

**重要：請嚴格遵循以下語言規則**

1. **Claude.md 內容**：使用zh-tw
2. **對話語言**：使用zh-tw
3. **程式碼註解**：使用zh-tw
4. **函數/變數命名**：使用en
5. **Git commit 訊息**：使用zh-tw
6. **文件字串 (docstrings)**：使用zh-tw
7. **專案文檔**：使用zh-tw
8. **其他發布用文件**：使用zh-tw

## 撰寫風格與格式

- **程式碼**（Python, Bash, etc.）、**配置檔案**（YAML, TOML, etc.）：遵循 ruff 規範。如果有不同會在後續 `uv run ruff check .` 檢查出異常。
- **專案文檔、說明文字、文件模板**：遵循 Google 風格。
- **Commit 與 PR 訊息**：遵循 Conventional Commit 格式與 Google 風格。
- **Changelog**：遵循 Keep a Changelog 格式。
- **分支名稱**：遵循 Conventional Branch Naming。

### Commit 撰寫規範

- **格式**：遵循 **Conventional Commits** 規範
- **風格**：Google 風格

#### 格式要求

```
<type>(<scope>): <description>    ← 第一行（50-72 字符）

[optional body]                   ← 詳細說明（72 字符換行）

[optional footer(s)]              ← 破壞性變更、問題參考
```

**重要說明**：
- **第一行**：GitHub 自動生成 release notes 使用
- **內容主體**：複雜變更的詳細解釋（不會出現在 release notes 中）
- **腳註**：破壞性變更和問題參考

### Pull Request 撰寫規範

**重要**：建立 PR 時必須遵循以下規範：

#### PR 標題格式
- 必須遵循約定式提交格式：`<type>(<scope>): <description>`
- 範例：`feat: add async operations with progress callbacks`

#### PR 內容格式
- 參考 `.github/pull_request_template.md` 中的模板
- 包含完整的變更說明、測試資訊、檢查清單

#### PR 標籤
- 根據 PR 標題自動分類（Release Drafter 自動處理）
- 確保選擇正確的變更類型

#### PR 描述要求
- 清楚描述變更內容和原因
- 列出相關的測試項目
- 確認所有檢查清單項目

**模板位置**：`.github/pull_request_template.md`
**風格**：Google 風格

### CHANGELOG 撰寫規範

**重要**：CHANGELOG.md 必須遵循 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) 格式、業界最佳實務與 **Google 風格**。

#### 基本原則
1. **面向用戶**：描述功能影響，而非技術實現細節
2. **語義化描述**：歸納整理變更，避免直接複製 commit 訊息
3. **完整版本記錄**：包含所有版本（包括 pre-release）
4. **標準分類**：僅使用 Keep a Changelog 的六個類別
   - **Added**：新功能
   - **Changed**：現有功能變更
   - **Deprecated**：即將移除的功能
   - **Removed**：已移除的功能
   - **Fixed**：錯誤修復
   - **Security**：安全性修復

**注意**：不使用 Conventional Commit 的分類（feat, docs, refactor 等），因為 CHANGELOG 面向最終用戶。

#### 版本處理策略
- **Stable Release (1.0.0)**：詳細描述所有重要變更，面向最終用戶
- **Pre-release (alpha, beta, rc)**：簡化描述，重點說明階段目標和主要改進
- **版本順序**：最新版本在上，按時間倒序排列
- **日期格式**：使用 YYYY-MM-DD 格式

#### 內容撰寫要求
- **用戶導向**：描述用戶能感受到的變化和價值
- **簡潔明確**：每項變更一行，易於掃讀
- **粗體標題**：使用 **功能名稱** 突出重要特性
- **避免技術細節**：不包含 commit hash、內部重構、開發工具變更等

#### 範例格式
```markdown
## [1.0.0] - 2025-08-01

### Added
- **Windows Filename Compatibility**: Automatic sanitization of problematic filenames
- **Enhanced Security Features**: Built-in protection against ZIP bombs

### Changed
- **API Architecture**: Redesigned for better performance and maintainability

### Fixed
- **Cross-Platform Compatibility**: Resolved Windows-specific path issues
```

### Release Notes 撰寫規範

**重要**：GitHub Release Notes 與 CHANGELOG 共用 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) 分類規範。

#### 分類與 Emoji 對應

| 類別 | Emoji | 說明 |
|------|-------|------|
| Added | 🚀 | 新功能 |
| Changed | 🔄 | 現有功能變更、資料更新 |
| Deprecated | ⚠️ | 即將移除的功能 |
| Removed | 🗑️ | 已移除的功能 |
| Fixed | 🐛 | 錯誤修復 |
| Security | 🔒 | 安全性修復 |

**注意**：不使用 Conventional Commit 的分類（feat, fix, chore 等），因為 Release Notes 面向最終用戶。

#### 範例格式
```markdown
# What's Changed

## 🐛 Fixed

- 修正 nightly 動態 tag 與分支命名衝突問題，確保自動更新流程穩定執行

## 🔄 Changed

- 更新泰國地理資料（2026-01-29、2026-02-13）

---

**完整變更記錄**: [v2.2.1...v2.2.2](https://github.com/RxChi1d/immich-geodata-zh-tw/compare/v2.2.1...v2.2.2)

**發布日期**: 2026-02-13
```

## 專案概述

本專案為 Immich 提供反向地理編碼功能的臺灣特化優化，旨在提升地理資訊的準確性及使用體驗。主要功能包括：

中文化處理：將國內外地理名稱轉換為符合臺灣用語的繁體中文。
行政區優化：解決臺灣直轄市與省轄縣市僅顯示地區名稱的問題。
提升臺灣資料準確性：利用中華民國國土測繪中心 (NLSC) 的官方圖資處理臺灣地區的地理名稱與邊界資料，確保數據來源的權威性。

- **Python 支援版本**：Python >= 3.13
- **Python 套件管理工具**：uv
- **uv 管理的虛擬環境**：`.venv/`

## 版本管理

本專案使用兩種獨立的版本號：

- **專案版本**（如 0.1.0）：遵循語義化版本規範，代表功能、API、錯誤修復的變更。專案版本變更時需更新發布說明、CHANGELOG 等面向用戶的文檔。
- **快取 Schema 版本**（如 1.0）：定義於 `TranslationCacheStore.VERSION`，僅在快取資料結構不相容時升級。快取版本幾乎不變動（可能整個 1.x 專案生命週期都維持 schema 1.0）。

**規則**：兩者獨立管理，專案版本升級不會影響快取 schema 版本，避免不必要的快取重建。

## 架構說明

### 地理資料處理流程（ETL 模式）

本專案採用 **Extract-Transform-Load (ETL)** 模式處理地理資料：

```
core/geodata/
├── base.py         # GeoDataHandler 抽象基類
├── taiwan.py       # TaiwanGeoDataHandler
└── japan.py        # JapanGeoDataHandler
```

每個國家的 Handler 包含三個階段的方法：

#### 1. Extract（提取）
- **方法**：`extract_from_shapefile(shapefile_path, output_csv)`
- **功能**：從 Shapefile 提取資料並轉換為標準化 CSV
- **輸入**：原始 Shapefile 檔案
- **輸出**：`meta_data/{country}_geodata.csv`
- **處理內容**：
  - 讀取 Shapefile（使用 geopandas）
  - 計算多邊形中心點（使用適當的投影）
  - 轉換為 WGS84 座標
  - 輸出標準化欄位：latitude, longitude, country, admin_1-4

#### 2. Transform（轉換）
- `convert_to_cities_schema(csv_path)`: 將標準 CSV 轉成 CITIES_SCHEMA，負責生成 geoname_id、對應行政區與補齊時區、國家代碼。

#### 3. Load（載入）
- `replace_in_dataset(input_df, base_geoname_id)`: 以新資料覆蓋 cities500 中對應國家的紀錄並回傳更新結果。國家代碼由 Handler 實例的 `COUNTRY_CODE` 類別變數自動決定。

### 常用指令

```bash
python main.py extract --country TW --shapefile <path_to_tw_shapefile>
python main.py extract --country JP --shapefile <path_to_jp_shapefile>
python main.py enhance --country-code <country_code>
```

### 在程式中使用

```python
from core.geodata import get_handler

handler = get_handler("TW")()
handler.extract_from_shapefile("path/to/file.shp", "output.csv")
df = handler.convert_to_cities_schema("output.csv", base_geoname_id=92_000_000)
updated_df, max_id = handler.replace_in_dataset(cities500_df, base_geoname_id=92_000_000)
```

### 擴充新國家


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RxChi1d/immich-geodata-zh-tw](https://github.com/RxChi1d/immich-geodata-zh-tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
