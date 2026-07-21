---
trigger: always_on
description: 給 AI coding agent（Claude Code、Cursor、Codex、Gemini Code Assist 等）使用的專案說明文件。人類貢獻者請優先閱讀 [`README.md`](./README.md)。
---

# AGENTS.md

給 AI coding agent（Claude Code、Cursor、Codex、Gemini Code Assist 等）使用的專案說明文件。人類貢獻者請優先閱讀 [`README.md`](./README.md)。

## 專案簡介

**高科校務通（NKUST AP）** 是高雄科技大學的校務系統 App，使用 Flutter 開發，支援 Android、iOS、macOS、Windows、Linux 五個平台（Web 因學校阻擋高請求 IP 已停用）。Repo 在 [`NKUST-ITC/NKUST-AP-Flutter`](https://github.com/NKUST-ITC/NKUST-AP-Flutter)。

由於學校沒有正式 API，整支 App 大量倚賴 **client-side web scraping**：對 `webap.nkust.edu.tw`、`stdsys.nkust.edu.tw`、`leave.nkust.edu.tw`、`vms.nkust.edu.tw` 等校內服務發 HTTP 請求並 parse HTML。這個爬蟲層是專案的核心複雜度來源。

## 技術棧

| 工具 | 版本 | 來源 |
|------|------|------|
| Flutter | `3.41.7` | `mise.toml` / `.fvmrc` |
| Dart SDK | `>=3.6.0 <4.0.0` | `pubspec.yaml` |
| Java | `zulu-21` | `mise.toml`（Android build） |
| Ruby | `3.3` | `mise.toml`（Fastlane） |

工具版本統一由 [mise](https://github.com/jdx/mise) 管理。Agent 在執行任何 build/test 命令前，請先確認當前 Flutter 版本符合 `mise.toml`，不要擅自升級 SDK。

主要相依：
- `ap_common` 系列：校務通跨校共用 UI / 函式庫
- `nkust_crawler`（in-repo package，路徑 `packages/nkust_crawler/`）：純 Dart 爬蟲套件
- `dio` + `cookie_jar` + `native_dio_adapter`：HTTP 與 cookie 管理
- `slang` / `slang_flutter`：i18n（設定見 `slang.yaml`）
- `flutter_inappwebview`：登入流程的 WebView fallback
- `syncfusion_flutter_pdf`：課表 / 證件 PDF 產生

## Repo 結構

```
nkust_ap/
├── lib/
│   ├── api/              # Scraper helpers、capability、registry、exceptions
│   ├── pages/            # UI 頁面（bus / info / leave / study 等）
│   ├── config/           # constants
│   ├── extensions/       # Dart extension methods
│   ├── integrations/     # Firebase、Google Sign-In 等外部整合
│   ├── l10n/             # slang i18n 來源檔（.i18n.json）與 generated/
│   ├── res/              # 顏色、樣式、theme
│   ├── utils/            # 工具函式
│   ├── widgets/          # 共用 widget
│   ├── app.dart          # MaterialApp
│   ├── main.dart         # entry point + flavor bootstrap
│   └── firebase_options.dart   # 由 flutterfire CLI 產生
│
├── packages/
│   └── nkust_crawler/    # 純 Dart 爬蟲套件（webap / stdsys / leave / vms-bus）
│
├── test/                 # 主 app 測試（hermetic）
├── assets/               # 圖片、CA、學程資料 JSON
├── assets_test/          # 測試用 HTML fixture
├── docs/                 # 架構 / 重構 / 遷移文件（見下方）
├── scripts/              # dev_configs encrypt/decrypt 腳本
├── .github/workflows/    # CI/CD
└── android/ ios/ macos/ linux/ windows/   # 各平台 native 殼
```

## 重要參考文件

修改特定模組前，先讀對應的 doc：

| 文件 | 內容 |
|------|------|
| `docs/crawler-architecture.md` | 爬蟲分層、ScraperRegistry、Capability 介面、Session 狀態機 |
| `docs/refactor-scraper-state-design.md` | Scraper 狀態設計決策記錄（ADR 風格） |
| `docs/cookie-handling.md` | Cookie / `SafeCookieManager` 行為 |
| `docs/changelog-pipeline.md` | PR → `changelog.json` → Fastlane 變更紀錄管線 |
| `docs/extracting-flutter-crawler-as-dart-package.md` | 將爬蟲抽離成獨立 Dart package 的流程（reusable guide） |
| `docs/migration-ap-common-v2.md` | ap_common v2 遷移筆記 |
| `docs/POLICY.md` | 隱私 / 資料處理政策 |

## 常用指令

### 環境設定

```bash
mise trust && mise install    # 安裝對應版本的 Flutter / Java / Ruby
flutter pub get               # 主 app
(cd packages/nkust_crawler && dart pub get)
```

### Lint / Test

```bash
# 主 app
flutter analyze --no-fatal-infos --no-fatal-warnings
flutter test

# 爬蟲 package（CI 是分開跑的，本地也請分開）
cd packages/nkust_crawler
dart analyze
dart test
```

**注意**：root 的 `analysis_options.yaml` 已將 `packages/**` 排除（見檔內註解）。Agent **不要**嘗試從 root 跑 `flutter analyze` 涵蓋 `packages/`，會因為 sub-package 自己的 dev_dependencies 沒被解析而炸掉。

### Live integration test（打真實 NKUST server）

`packages/nkust_crawler/test/` 內有以 `@Tags(['integration'])` 標記的 live test：

```bash
cd packages/nkust_crawler
NKUST_USERNAME=xxx NKUST_PASSWORD=xxx dart test --tags integration
# 可選：開 HTTP log
NKUST_HTTP_LOG=1 dart test --tags integration
```

沒有環境變數時這些 test 會 skip（見 `dart_test.yaml`）。Agent 不要把帳密寫進 commit 或 log。

### Code generation

```bash
flutter pub run build_runner build --delete-conflicting-outputs    # json_serializable + slang
```

修改 `*.i18n.json` 或加 `@JsonSerializable` 的 model 後要跑一次。

### Run / Build

```bash
flutter run -d macos        # 或 chrome / windows / linux / <device-id>
flutter build apk --release
flutter build appbundle --release
flutter build ipa --release
flutter build macos --release
```

正式發版會帶 `--build-number=<VERSION_CODE>`，由 CI 控制，本地不用手動指定。

## 爬蟲層核心概念

> 摘要自 `docs/crawler-architecture.md`，動到 `lib/api/` 或 `packages/nkust_crawler/` 前必讀原始文件。

分層（上到下）：

```
UI (lib/pages/*)
  → Helper Facade (lib/api/helper.dart)
    → ScraperRegistry (lib/api/scraper_registry.dart)
      → Capability 介面（6 個 *_provider.dart）
        → 各 ScraperHelper（WebAp / Stdsys / Leave / VmsBus / NKUST）
          → 對應 Parser
```

- **Helper Facade** 是 UI 唯一入口，不要從 UI 直接 import 個別 scraper helper
- **Capability provider** 用來宣告「某個 helper 提供哪些功能」（成績、課表、請假…），Registry 會挑能滿足的 helper
- 多個學校系統可能都能查同一資料（例如成績），Registry 的選擇順序見原 doc
- **Session 狀態機**處理登入 → cookie 失效 → 重登的流程，不要繞過
- 例外階層在 `lib/api/exceptions/`，新增錯誤類型請延伸既有 base class

## 目標伺服器（爬蟲對象）

App 沒有自建後端，所有資料來自爬學校的五個網域。每個網域有自己的登入機制、cookie 範圍、HTML 結構，動到任何一個 helper 前先看清楚是哪一隻。

| 網域 | Helper | Dio | CookieJar | 登入方式 | 提供能力 |
|------|--------|-----|-----------|---------|---------|
| `webap.nkust.edu.tw` | `WebApHelper` | own | own（**shared**） | captcha + form POST → `perchk.jsp` | Course / Score / UserInfo / Semester |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NKUST-ITC/NKUST-AP-Flutter](https://github.com/NKUST-ITC/NKUST-AP-Flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
