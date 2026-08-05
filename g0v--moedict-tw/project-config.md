---
trigger: always_on
description: 萌典（moedict）字典查詢網站的現行主站程式碼：React + TypeScript + Vite 前端、
---

# AGENTS.md — cf-moedict-webkit-neo（moedict.tw）

萌典（moedict）字典查詢網站的現行主站程式碼：React + TypeScript + Vite 前端、
Cloudflare Workers 後端、R2 儲存。本檔是給 AI agent 與新進開發者的工作手冊——
內容以「未來 session 直接可用」為準，改動架構時請同步更新。

## 專案定位（多個 surface，別搞混）

| Surface            | 是什麼                                                   | 原始碼                           |
| ------------------ | -------------------------------------------------------- | -------------------------------- |
| **www.moedict.tw** | 現行主站：本 repo 的 Worker `cf-moedict-webkit-neo` + R2 | 本 repo                          |
| www.moedict.org    | 凍結的舊版靜態前端（GitHub Pages），只收安全性修正       | `~/w/moedict-webkit`（gh-pages） |
| 行動 App           | Capacitor 離線版，資料由本 repo `data/dictionary` 供給   | `~/w/moedict-app`                |

- 裸網域 `moedict.tw` 會 301 到 `www.moedict.tw`——線上驗證一律打 `www`。
- 舊版 `~/w/moedict-webkit` 的 `view.ls` 是 UI 行為的 ground truth：
  移植或修 UI 迴歸時先讀它，不要憑猜測（例：`h1.title` 內的 DOM 順序）。

## 語言代碼（四本字典）

| 代碼 | 字典               | Hash 前綴 | 路由 | pack 目錄 |
| ---- | ------------------ | --------- | ---- | --------- |
| `a`  | 華語（國語辭典）   | `#`       | `/`  | `pack/`   |
| `t`  | 臺灣台語（閩南語） | `#'`      | `/'` | `ptck/`   |
| `h`  | 臺灣客語           | `#:`      | `/:` | `phck/`   |
| `c`  | 兩岸詞典           | `#~`      | `/~` | `pcck/`   |

（舊版 CLAUDE.md 此表有誤，以上為正確對應。）

## 技術棧與目錄結構

- 前端：React 19、TypeScript、Vite+（Vite 8）、react-router-dom v7
- 後端：Cloudflare Workers（`worker/index.ts`，經 `@cloudflare/vite-plugin` 建置）
- 儲存：R2（`FONTS` / `ASSETS` / `DICTIONARY` 三個 binding）
- 工具鏈：**Vite+** 統一 dev/build/test/lint/format/runtime/package-manager 入口；
  `vite.config.ts` 同時持有 Vite、Vitest projects、Oxlint 與 Oxfmt 設定。
- 套件管理：Vite+ 管理的 **Bun**（`vp install`、`vp run <script>`；lockfile
  是 `bun.lock`）。

```
src/
  pages/        # DictionaryPage（核心詞條頁）、Dictionary-a/c/h/t、ListView、
                # RadicalView、RadicalDetailView、StarredPage、About、Privacy
  components/   # Layout、navbar、searchbox、sidebar、StrokeAnimation、
                # CharacterImageView、AssetLoader、InlineStyles、user-pref
  api/          # Worker 端 API handler（handleDictionaryAPI、handleLookupAPI、
                # handleListAPI、cache.ts 快取政策）
  oembed/       # /embed/<詞> 卡片 + /api/oembed
  ssr/          # head.ts（伺服端 <head> 注入）
  hooks/ utils/ # decorateRuby、pinyin 轉換、dictionary-route 等
worker/
  index.ts      # Worker 入口 + dispatch() 路由表
data/
  dictionary/   # 字典資料（真實來源，上傳至 R2；見「字典資料格式」）
  assets/       # 舊版前端資產（styles.css、字型、JS；上傳至 R2）
scripts/        # build-search-index、build-pinyin-lookup、merge-coverage 等
commands/       # upload_dictionary.sh、upload_assets.sh、fetch-moe-stroke.mjs、sync-moe-stroke-corpus.mjs
tests/          # unit / integration / e2e 三層
memory/MEMORY.md  # 跨 session 的架構筆記（與本檔互補）
```

## 常用指令

```bash
vp install                    # 依 devEngines.packageManager 安裝相依
vp run dev                    # predev 重建索引，再啟動 Vite + Miniflare
vp run build                  # prebuild 重建索引、tsc -b，再執行 vp build
vp check                      # Oxfmt + type-aware Oxlint + TS diagnostics
vp run typecheck              # canonical tsc -b --noEmit project build

vp test                       # unit + integration 兩個 Vitest project
vp run test:unit              # happy-dom unit tests
vp run test:integration       # Miniflare 實體 Worker API 測試
vp run test:e2e               # Playwright：chromium project（全部）+ webkit-romanization project（@romanization 聚焦測試）
vp run test:e2e:visual        # 視覺回歸（chromium only；baseline 僅 commit linux 版）
vp run test                   # unit + integration + e2e 三層全跑
vp run test:coverage          # 三層 coverage 合併至 coverage/combined/
```

`vp dev` / `vp build` 是不可覆寫的 Vite+ built-in，不會執行本專案的
`predev` / `prebuild` 與額外的 `tsc -b`；日常開發、正式 build 與部署一律用
`vp run dev` / `vp run build`。Vitest 可直接用 `vp test`；要限定層級則用
`vp run test:unit` / `vp run test:integration`。不要用裸 `bun test`——它不讀
`vite.config.ts` 的 happy-dom、setup、alias 與 project 設定。

## 部署（零停機兩階段 rollout，這是規範不是建議）

**沒有裸 `wrangler deploy` 這條路。** 標準指令一律走安全 orchestrator：

```bash
bun run deploy:staging   # 先 staging：build → 發布 R2 → 兩階段 rollout
# → 自動於 https://cf-moedict-webkit-neo-staging.audreyt.workers.dev 做 0%/100% smoke + 120 秒 probe
bun run deploy           # staging 通過後才部署 production；同樣 build → 發布 R2 → rollout
```

`deploy`/`deploy:staging` 都是「同一次 build 產物」貫穿到底的三段 `&&` 鏈：
`env -u CLOUDFLARE_ENV vp run build && env -u CLOUDFLARE_ENV node scripts/release-publish.mjs && env -u CLOUDFLARE_ENV node scripts/release-deploy.mjs`
（production 每段都用 `env -u CLOUDFLARE_ENV` 明確清掉環境變數，讓 production
絕不會被外層 shell/CI 殘留的 `CLOUDFLARE_ENV=staging` 汙染，fail-closed 而非
沿用繼承值；staging 則是三段各自帶 `CLOUDFLARE_ENV=staging` 前綴。兩者都是
因為 `&&` 串接的每個子命令是各自獨立的行程，環境變數前綴不會跨命令繼承）。
**絕不能在 publish
與 rollout 之間夾第二次 build**——那會讓 `release-deploy.mjs` 內部重新算出的
release manifest／digest 與剛剛實際發布到 R2 的那份不一致。完整協定見
[`notes/零停機部署筆記.md`](./notes/零停機部署筆記.md)。

- **兩階段 rollout**：`release-deploy.mjs` 用 `wrangler versions upload/deploy`
  做 new0%/old100% → 30 秒初始 propagation sleep → override smoke（若只看見
  已知舊版 release，逐路由最多再 poll 6 次、每次 10 秒；非 200、缺/異常標頭或
  第三個 release 立即 fail-closed）→ new100%/old0% → 30 秒初始 propagation
  sleep → ≥120 秒 continuous probe。continuous probe 只允許「已知舊版 release」在
  單一 60 秒 settling grace 內短暫出現；任何舊版 sighting 都會把健康 soak 歸零，
  之後仍必須重新累積完整 ≥120 秒新版健康結果。60 秒 grace 到期後仍看見舊版、或

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [g0v/moedict.tw](https://github.com/g0v/moedict.tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
