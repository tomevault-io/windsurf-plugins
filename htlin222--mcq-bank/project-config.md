---
trigger: always_on
description: 專科考試共筆題庫(hema-2026)。React + Vite 前端在 Cloudflare Pages,Hono Worker
---

# CLAUDE.md

專科考試共筆題庫(hema-2026)。React + Vite 前端在 Cloudflare Pages,Hono Worker
在 `/api/*`,D1 / R2 / Workers AI / Durable Objects 全在免費額度內,登入交給
Cloudflare Access(Zero Trust),**應用層沒有任何 auth 程式碼**。

這份檔案分兩半:

| 區                                     | 讀的時機                                                    |
| -------------------------------------- | ----------------------------------------------------------- |
| **§0–§7 上手區**(本頁前段)             | 開機、設定、部署、動資料庫、跑測試 —— 每個 session 都會用到 |
| **設計筆記**(`## Project Overview` 起) | 只在動到那個功能時讀對應那一節;每節標題就是它的主題         |

---

## §0 三十秒地圖

| 層       | 技術                                                                         | 在哪                                         | 本機埠             |
| -------- | ---------------------------------------------------------------------------- | -------------------------------------------- | ------------------ |
| 前端     | React 18 · Vite · TailwindCSS · TipTap · PWA(`injectManifest`)               | `frontend/src/`                              | 5173               |
| API      | Hono on Workers,`nodejs_compat`                                              | `worker/`                                    | 8787               |
| 資料     | D1(SQLite)`migrations/`,R2 圖片與 PDF,3 個 SQLite DO(聊天、跨裝置狀態、2048) | `worker/{chat-room,user-state,play-2048}.ts` | `.wrangler/state/` |
| 登入     | CF Access 注入 `Cf-Access-Jwt-Assertion`;本機用 `X-Dev-Email` 繞過           | `worker/lib/auth.ts`                         | —                  |
| 腳本     | 匯入 / 部署 / 同步名單,全部從 `config.toml` 讀名字                           | `scripts/`                                   | —                  |
| 資料管線 | 原始考題與批次 JSON(gitignored),`years/<民國年>/batches/*.json` → CSV → D1   | `years/`(本機才有)                           | —                  |

**Vite proxy 把 `/api`、`/img`、`/pdf` 轉到 8787 並注入 `X-Dev-Email`**(值來自
`config.toml [dev].dev_email`),Worker 在 `CF_ACCESS_TEAM_DOMAIN === 'localhost'` 時
接受它。這就是整個本機登入機制。

---

## §1 環境檢查

### 1.1 工具

```bash
node -v            # ≥ 20(scripts 用 --experimental-strip-types 直接跑 .ts)
pnpm -v            # 唯一的套件管理器;不要用 npm install
python3 --version  # ≥ 3.11(scripts 用 stdlib tomllib 讀 config.toml)
wrangler --version # ≥ 4;deploy 與所有 d1/r2 指令都靠它
uv --version       # 只有 anki:build 與 bank-ingest skill 用到
```

### 1.2 四份 gitignored 的設定檔

**沒有這四份,`pnpm dev` 起不來或登不進去。** 這是新 clone 最常卡住的地方。

| 檔案            | 誰產生                                                                | 誰讀                                                                           | 沒有它的症狀                                                                            |
| --------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| `config.toml`   | `./scripts/setup.sh`(從 `config.example.toml`)                        | 所有 scripts、`frontend/vite.config.ts`(build 時注入 `__APP_CONFIG__`)         | vite 啟動就炸:`ENOENT config.toml`                                                      |
| `wrangler.toml` | `setup.sh`(從 `wrangler.example.toml`);`deploy.sh` 回填 `database_id` | wrangler                                                                       | `wrangler dev` 找不到 bindings                                                          |
| `.env`          | `setup.sh`(從 `.env.example`)                                         | `deploy.sh`、`sync-access.ts`、`setup-public-bypass.sh`、`backfill-vectors.ts` | 只影響部署類腳本,本機開發不需要                                                         |
| `.dev.vars`     | **手動** `cp .dev.vars.example .dev.vars`(setup.sh 不管它)            | `wrangler dev` 當 Worker env                                                   | 所有 `/api/*` 回 `401 unauthenticated` —— 因為 `CF_ACCESS_TEAM_DOMAIN` 不是 `localhost` |

一鍵檢查:

```bash
ls config.toml wrangler.toml .env .dev.vars            # 四份都要在
grep -c REPLACE_ME wrangler.toml                        # 0 = database_id 已回填(本機 dev 不需要,remote 才需要)
grep CF_ACCESS_TEAM_DOMAIN .dev.vars                    # 必須是 localhost
node scripts/lib/cfg.mjs project.d1_db                  # 印得出 db 名 = config.toml 讀得到
lsof -iTCP:8787 -sTCP:LISTEN                            # 有東西 = 埠被佔(見 1.3)
```

### 1.3 埠 8787 常被別的東西佔走

本機「所有 API 都 500 / 404」多半不是程式問題,是 OpenEvidence MCP 的 relay daemon
之類佔了 8787,wrangler 根本沒在聽。換埠時兩邊要一起換:

```bash
WORKER_PORT=8788 pnpm dev -- --port 8788      # 終端 A(vite 讀 WORKER_PORT 跟著轉)
WORKER_PORT=8788 pnpm --dir frontend dev      # 終端 B
```

---

## §2 本機啟動

```bash
pnpm install && pnpm --dir frontend install
cp .dev.vars.example .dev.vars                 # 首次
pnpm db:migrate:local                          # 建 schema + 範例題(0002 / 0004 的種子列)
pnpm dev                                       # 終端 A:gen:bundles → wrangler dev :8787
pnpm --dir frontend dev                        # 終端 B:vite :5173
```

啟動後的驗證,三條由外到內:

```bash
curl -s localhost:8787/api/health                                 # {"ok":true,...} — Worker 活著,不需 auth
curl -s -H 'X-Dev-Email: you@example.com' localhost:8787/api/me   # 回 JSON 使用者 = dev 繞過生效
open http://localhost:5173                                        # 首頁 dashboard、右上角有 dev_email 的頭像
```

**本機資料有兩種來源,選一種:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htlin222/mcq-bank](https://github.com/htlin222/mcq-bank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
