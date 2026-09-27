---
trigger: always_on
description: Architecture reference for AI agents working on **Exameow**. Read this before exploring the codebase.
---

# AGENTS.md

Architecture reference for AI agents working on **Exameow**. Read this before exploring the codebase.

## What Exameow Is

AI-powered exam question generator. Users upload study materials (PDF, DOCX, XLSX, PPTX, EPUB, ODT, TXT, CSV, HTML) and get exam questions generated via any OpenAI-compatible API. Includes a built-in practice/quiz mode with wrong-question tracking. Exports to XLSX/CSV.

Version `1.2.1` (kept in sync across root `package.json`, `src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`, `workers/package.json`).

## Release Rules

- **版本号语义（semver）**：第一位 = 不兼容的大更新；第二位 = 新功能；第三位 = Bug 修复。
- **Bump 版本时**：同步改 4 个文件 + `Cargo.lock` 中 `name = "exameow"` 条目（**只改 exameow 条目，千万别全局替换**——`cesu8` 等依赖锁版本也是 x.y.z，误改会导致全平台构建失败）。
- **发布流程**：bump 提交 → 打 `v*` tag 推送触发 CI（desktop/mobile/docker 三条流水线）→ CI 生成的 GitHub Release **默认是草稿，必须发布（`gh release edit vX.Y.Z --draft=false`），否则 Tauri 更新器看不到 `latest.json`** → 用 `bash scripts/deploy-cf.sh` 顺便更新 Cloudflare 线上版。
- **移动端 OTA 热更新**：`src-tauri/src/ota.rs` 自研实现（assets 替换 + 三态回滚 staged→booting→committed），仅 Android/iOS 生效，桌面端仍用官方 updater。CI 随 release 附加 `mobile-dist.tar.gz` + `mobile-ota.json`；App 查 `releases/latest/download/mobile-ota.json`。**若某版本前端依赖新增的原生能力（Rust 命令/插件），发版前必须把仓库根 `ota.json` 的 `minShell` 提高到能支持它的最低 APK 版本**，否则旧壳会热更到不兼容的前端，调新命令时报 `command xxx not found`（v1.3.5 真实事故：`explain_question` 新增但 minShell 滞留 1.3.0，旧壳热更后 AI 解析全挂；且已中招设备无法靠 OTA 自愈——minShell 只在下载决策时校验，必须重装新 APK）。纯前端修复无需动 `minShell`。**防忘**：mobile CI 首步跑 `scripts/check-ota-minshell.sh`，自动 diff 前端 `invoke()` 命令集与上一 tag 原生代码（`src-tauri/` + `plugins/`），发现新命令但 `minShell` 未提到 ≥ 当前版本则流水线直接失败。

**章节功能发布提醒**：`auto_chapter` / `chapter_names` 依赖本次新增的 Rust 生成提示词能力。下次发布此功能时须升级次版本，并将 `ota.json` 的 `minShell` 提高到包含该能力的新壳版本；旧 1.5.0 壳会忽略这些参数，不能仅靠新命令检测覆盖此兼容性变化。

## Tech Stack

- **Frontend**: Vue 3 + Vite + Pinia + Vue Router + TypeScript, Tailwind CSS 3.4 (custom Material You tonal palette)
- **Desktop/Mobile**: Tauri v2 (Rust shell)
- **Self-hosted backend**: Rust / Axum 0.8
- **Serverless backend**: Cloudflare Workers (Hono 4.7)
- **Shared core logic**: Rust crate `exameow-core` (parsing, AI client, exam gen, export, encrypted config)
- **Package mgmt**: pnpm workspace + Cargo workspace (monorepo)

## Three-Backend Architecture (KEY CONCEPT)

The **same Vue frontend** targets three interchangeable backends. `frontend/src/api/index.ts` auto-detects the platform at runtime and routes accordingly:

| Platform | Detector | API module | Backend |
|----------|----------|-----------|---------|
| Tauri desktop/mobile | `isTauri()` | `api/bridge.ts` → `invoke()` | `src-tauri/src/lib.rs` (Tauri commands) |
| Cloudflare | `isCloudflare()` | `api/cf.ts` → `fetch()` | `workers/src/index.ts` (Hono) |
| Web / Docker | fallback | `api/http.ts` → `fetch()` | `packages/server` (Axum) |

Platform detection lives in `frontend/src/utils/platform.ts`.

**Important consequence**: Core logic (file parsing, prompt building, export) is **duplicated** in Rust (`packages/core`) and TypeScript (`workers/src/*`, `frontend/src/utils/*`). When changing generation prompts, parsing, or export format, update BOTH the Rust and TS implementations to keep parity.

## Directory Map

```
frontend/              Vue 3 SPA (hash routing)
  src/
    api/               Platform-routed API layer (index.ts dispatches to bridge/http/cf)
    stores/            Pinia: exam.ts, practice.ts, config.ts, wrongQuestions.ts, fileInput.ts, i18n.ts
    views/             GenerateView, PracticeView, ConfigView, PreviewView
    utils/             Browser-side: fileParser, pdfParser, importParser, aiClient, platform
    components/        config/ generate/ layout/ practice/ preview/
  dist/                Build output (served by Axum or copied to workers/public)

packages/
  core/                Rust crate `exameow-core` (shared server logic)
    src/parser/        parse_file() dispatch → pdf/docx/pptx/excel/csv/epub/odt/html/txt
    src/ai/            OpenAI-compatible HTTP client (client.rs)
    src/exam/          types.rs (Question/ExamParams), prompt.rs (generate_exam + prompts)
    src/export/        writer.rs (CSV), xlsx.rs (manual ZIP+XML, no lib)
    src/config/        store.rs (AES-256-GCM encrypted config persistence)
  server/              Axum HTTP server; routes.rs (AI 端点) + relay.rs (在线考试,SQLite via rusqlite)
  shared/              TS shared types (@exameow/shared) src/types.ts

src-tauri/             Tauri app; src/lib.rs = all Tauri commands; tauri.conf.json; capabilities/
workers/               Cloudflare Worker; src/{index,ai,exam,parser,export,types,relay}.ts; wrangler.toml; migrations/ (D1)
                       relay.ts = exam publish/take relay (D1 EXAM_DB, cron cleanup; /api/exam/* routes)
scripts/               deploy-cf.sh, docker-build.sh, start-android-emulator.sh, check-ota-minshell.sh
.github/workflows/     release-desktop.yml / release-mobile.yml / release-docker.yml (v* tag 触发;Docker 推 Docker Hub `ailm32442/exameow`)
```

## Commands

| Task | Command |
|------|---------|
| Install deps | `pnpm install` |
| Frontend dev | `cd frontend && pnpm dev` (port 5273) |
| Frontend build | `cd frontend && pnpm build` |
| Frontend typecheck | `cd frontend && pnpm run type-check` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heshengtao/exameow](https://github.com/heshengtao/exameow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
