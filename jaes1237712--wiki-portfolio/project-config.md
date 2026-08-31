---
trigger: always_on
description: > 這份檔案是**跨裝置的單一真實來源**。換一台電腦、隔一段時間回來、或開一個新的 Claude Code
---

# CLAUDE.md — wiki-portfolio 開發指南

> 這份檔案是**跨裝置的單一真實來源**。換一台電腦、隔一段時間回來、或開一個新的 Claude Code
> session,都先讀這份。進度、決策、待辦都記在這裡,不要只留在對話裡。
>
> **每完成一個 Phase 或做出新決策,就更新本檔的「目前進度」與「決策紀錄」。**

## 專案是什麼

對繁體中文維基百科連結網絡的互動式分析作品集網站:爬取條目連結圖 → Infomap 社群偵測 →
瀏覽量活化(異常偵測/趨勢)→ 語意搜尋(pgvector),前端用 Sigma.js 呈現整張拓樸圖。

功能的優先序就是上面的順序(決策 24):**圖是骨架,瀏覽量的活化是產品價值所在,
語意搜尋是加分項**。Embedding 只掛在節點上,不參與決定圖的結構(決策 23)。

目的是展示全端能力(資料工程 + 後端 + 前端 + 架構),所以每一層都要做得像正式產品:
沒有寫死的密鑰、CORS 限制來源、有測試、有排程重算。

完整分階段計畫見 [docs/roadmap.md](docs/roadmap.md)。

> ⚠️ **2026-08-19:資料獲取策略正在重新設計中,先讀
> [docs/data-strategy.md](docs/data-strategy.md)。**
>
> 那份文件記錄了一次長討論的結果,其中包含一個會影響 Phase 3-8 的發現:
> **目前這張 13,085 節點的圖,封閉度只有 2.4%**(只有 314 個節點被展開過),
> 導致 hub/authority/bridge 這些角色指標實際上在測爬蟲軌跡而不是維基百科結構。
>
> 提議的方向是從「一張扁平大圖」改成「多個封閉的精緻小圖 + 推導出來的主題地圖」。
> **那些提議尚未定案**,文件裡每一節都有 [實測]/[提議]/[待驗]/[待決] 標記,
> 不要把 [提議] 當成已定案。已定案的東西才會進 decisions.md。
>
> 這個專案同時是**科學項目、工程項目、產品設計項目**,三個面向會互相推翻對方的
> 結論 —— 發生衝突時要記錄下來,不要偷偷選一個。

> ✅ **2026-08-19 稍晚新增五條已定案的決策(23-27),見 [docs/decisions.md](docs/decisions.md):**
>
> - **#23** Embedding 只掛在節點上(語意搜尋 + 節點特徵),**圖的邊與邊權重永遠只由真實超連結
>   決定**;明確排除 cosine 建圖/加權與 GraphRAG 路線。
> - **#24** 產品重心是**瀏覽量資料的活化**,語意搜尋降為加分項(取代 #20)。
> - **#25** **Phase 3(embedding)延後到 Phase 4-6 之後**,新順序 4 → 5/6 → 3。
>   ⚠️ 但模型/維度/`halfvec` 的選擇仍須在 Phase 4 建表前決定。
> - **#26** 瀏覽量**不抽樣**,所有收錄節點都抓(取代 data-strategy 5.4 的「只抽前 10%」)。
> - **#27** **每日更新整個刪掉**,只留每週 GitHub Actions(取代 #6 的每日 Cron)。
> - **#28** 瀏覽量做成**歷史資料產品**(2015-07 起十一年),不是即時儀表板。
>   實測:一個請求就能拿 **4,050 個日資料點**(現在只拿 365),請求數完全不變,
>   所以歷史深度是免費的。10,000 節點 → 約 **4,000 萬個資料點**,**一次抓完、不需排程**。
> - **#29** **完全取消排程重算**(取代 #6 全部內容、data-strategy 5.2、roadmap Phase 10)。
>   資料是一個固定快照,要更新就手動跑 pipeline。連帶讓「社群編號跨週漂移」與
>   「embedding 增量重算」兩個治理問題**直接消失**。
> - **#30** ~~條目誕生時間用首次修訂時間~~ —— **一半被 #31 作廢**。
>   `run-stage first-revision` 不做了;但 **`page_id` 仍要存**(穩定識別碼,title 會變它不會,
>   而且每個 API 回應本來就免費附帶)。
> - **#31** **產品是一個 dashboard,不是一篇策展好的分析。**
>   v1 就是「**使用者在圖上選範圍(節點/社群/主題)× 選時間區間 → 看趨勢圖**」,
>   異常改成對選定範圍**即時算**。所有「預先的應用」(異常擴散、結構與注意力落差、
>   季節性、條目成長史、AI 衰退敘事)**一律不做** —— 使用者要工具,不要我們的分析報告。
>   決策 28 的十一年資料照抓,因為範圍選擇器要有深度才有意義。
>
> ✅ **瀏覽量這條線的問題全部收完了。** 還開著的是與主題劃分有關的 **J/K/L/M**
> (卡在實驗 1)、以及 E/F/H。

## 技術棧(已定案,不要重新討論)

| 層面 | 選擇 |
|---|---|
| 資料管線 | Python 3.11+,離線批次(igraph + Infomap) |
| 資料庫 | Neon(Postgres + pgvector),Drizzle 管 schema |
| 後端 | Cloudflare Workers + Hono |
| Embedding | Cloudflare Workers AI(離線與線上共用同一模型) |
| 前端 | Vite + React SPA → Cloudflare Pages |
| 圖視覺化 | Sigma.js + graphology(react-sigma) |
| 圖表 | Recharts |
| Monorepo | **npm workspaces(不要用 pnpm/yarn)** |
| 排程 | 輕量每日:Cloudflare Cron Trigger;重量週期:GitHub Actions |

選型理由寫在 [docs/decisions.md](docs/decisions.md)。

## Repo 結構

```
wiki-portfolio/
├── apps/
│   ├── web/            # Vite + React SPA(Phase 6,尚未建立)
│   └── api/            # Cloudflare Worker + Hono(Phase 5,尚未建立)
├── packages/
│   ├── shared-types/   # ✅ 前後端共用型別
│   └── db-schema/      # ✅ Drizzle schema + embedding 模型設定
├── pipeline/           # 🚧 Python 離線 pipeline(骨架完成,階段實作見 Phase 1-4)
│   ├── wiki_pipeline/
│   ├── experiments/    # ✅ 探索性實驗腳本(data-strategy.md 的數字都是這裡跑的)
│   └── tests/
├── docs/
│   ├── roadmap.md      # 完整分階段計畫
│   ├── data-strategy.md # 🔴 資料獲取/邊界/儲存策略(進行中,先讀這份)
│   ├── decisions.md    # 決策紀錄 + 待決事項
│   └── legacy/         # 舊專案參考程式碼(唯讀)
└── .github/workflows/  # 重量重算排程(Phase 10)
```

## 新裝置設定步驟

```bash
git clone git@github.com:jaes1237712/wiki-portfolio.git && cd wiki-portfolio

# 1. Node 端(需要 Node 20+)
npm install
npm run build          # 建 packages/*
npm run typecheck

# 2. Python 端(需要 Python 3.11+)
cd pipeline
python -m venv .venv
source .venv/Scripts/activate   # Windows Git Bash;macOS/Linux 用 .venv/bin/activate
pip install -e ".[dev]"
pipeline show-config            # 確認設定讀得到

# 3. 環境變數(絕不進版控)
cp packages/db-schema/.env.example packages/db-schema/.env   # 填 Neon dev branch 連線字串
```

需要的雲端帳號:Neon(專案 + `dev`/`main` 兩個 branch)、Cloudflare(Workers / Pages / Workers AI)。

## 目前進度

| Phase | 內容 | 狀態 |
|---|---|---|
| 0 | Repo 骨架、npm workspaces、shared-types、db-schema | ✅ 完成(Neon 專案尚未開) |
| 1 | Pipeline:爬蟲 + 連結圖 + 條目簡介 + 每日瀏覽量 | ✅ 完成 |
| 2 | Pipeline:建圖 + 邊權重 + 兩層社群偵測(36 個測試) | ✅ 完成 |
| 3 | Pipeline:Workers AI embedding | ⬜ 未開始 ⚠️ **已延後到 5/6 之後**(決策 25) |
| 4 | Pipeline:寫入 Neon(**里程碑**,解鎖 5/6 平行開發) | ⬜ 未開始 ⚠️ schema 會改,見 data-strategy.md。**現在是下一個要動工的 Phase** |
| 5 | 後端 Worker(Hono) | ⬜ 未開始 |
| 6 | 前端 React app | ⬜ 未開始 |
| 7 | 小型資料集整合測試 | ⬜ 未開始 |
| 8 | 全規模 pipeline 執行(真實主題) | ⬜ 未開始 ⚠️ 爬取策略重新設計中 |
| 9 | 部署 | ⬜ 未開始 |
| ~~10~~ | ~~排程重算~~ | ❌ **已取消(決策 29)**,資料改成固定快照 |
| 11 | 作品集包裝(首頁) | ⬜ 未開始 |

**Phase 1 已完成的部分:** `wiki_pipeline/state.py`(SQLite checkpoint,取代舊版 5 個 JSON 快取檔)、
`wiki_api.py`(async + 併發上限 + 退避重試 + 簡繁/redirect 標題收斂)、`scrape.py`(逐層 BFS、
每批一個交易、可續傳、原子寫出 `wiki_network.json`)、`tests/`(respx 假 API,不打真的 Wikipedia)。
`zh.py`(OpenCC s2twp 顯示標題)。已對真實 API 實測:`圖論` depth 1 → 251 節點,
標題正確收斂(`圖論`→正式標題 `图论`)且顯示標題轉成台灣繁體(`计算机科学`→`電腦科學`),
續跑 0 次額外請求。

**Phase 2 完成的部分:** `graph_build.py`(records → igraph 有向圖,去自環/重複邊)、
`weighting.py`(拓樸相似度,**保留舊版非對稱性**,鄰居集合預先算好取代 O(E·deg) 重算)、
`communities.py`(igraph 內建 Infomap 兩層社群、有效子圖、hub/authority/center/bridge
特殊節點、社群關係 meta-graph)、`stages.py`(階段串接 + GraphML/JSON 中繼產物)。
CLI:`pipeline run-stage graph` 與 `run-stage communities`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaes1237712/wiki-portfolio](https://github.com/jaes1237712/wiki-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
