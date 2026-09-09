---
trigger: always_on
description: 本文件为 AI Agent 操作本仓库（`@lzwme/m3u8-dl`）提供指引。
---

# AGENTS.md

本文件为 AI Agent 操作本仓库（`@lzwme/m3u8-dl`）提供指引。

## 常用命令

- **安装依赖**：`pnpm install`。必须用 pnpm（workspaces），锁文件已 gitignore。
- **开发**：`pnpm dev` — 并发运行 `dev:sdk`（`tsc -p tsconfig.cjs.json --watch`）、`dev:server`（`nodemon bin/m3u8dl.js server --debug`）、`dev:frontend`（Vite 5173）。前端 dev 将 `/api`、`/ws` 等代理到 6600，需先起后端。
- **编译 SDK**：`pnpm build:cjs` — `src/**` → `cjs/`（CommonJS + d.ts）。改 `src/` 后必须重编译，CLI 与前端代理都加载 `cjs/` 产物。
- **全量构建**：`pnpm build` = `clean` → `build:cjs` → `build:frontend` → `build:capture`。前端输出到根 `client/`，油猴脚本输出到 `client/m3u8-capture.user.js`。
- **检查/格式化**：`pnpm lint`（根包 Biome）、`pnpm lint:all`（根 + capture + frontend）、`pnpm format`、`pnpm fix`、`pnpm fix:all`。
- **清理/发布/文档**：`pnpm clean`（依赖 `flh`）、`pnpm release`（build + standard-version）、`pnpm doc`（typedoc → `docs/`）。
- **CLI 调试**：`node bin/m3u8dl.js <url> --debug`、`node bin/m3u8dl.js server -p 6600 -t <token>`、`node bin/m3u8dl.js search -u <采集站api>`。
- **测试**：无单测框架，`pnpm test` 实际等于 `lint:all`。`test/test-video-parser.ts` 是遗留手动脚本，未被 tsconfig include 且签名已失同步，需自行用 `tsx` 运行并修正。

## 架构

### Monorepo

根包是核心 SDK + CLI（`main: cjs/index.js`，bin `m3u8dl`）。`packages/*`：`frontend`（Vue 3 + Pinia + Router + vue-i18n + Tailwind v4 + Vite）、`m3u8-capture`（油猴脚本，IIFE 单文件）、`portal`（静态着陆页）、`m3u8dl-app`（Electron）、`m3u8dl-electrobun` 与 `ai-agent`（非活跃）。

关键约定：**根 `client/` 同时是前端产物目录和后端 Express 静态目录**（`express.static(resolve(__dirname,'../../client'))`），所以 `build:frontend` 之后后端才服务到新 UI。

### 编译布局

源码只有 `src/`，产物 `cjs/`（target ESNext、module Node16、strict 但 `strictNullChecks:false`），文件名一一对应。`tsconfig.cjs.json` 是唯一编译入口。因 module 为 Node16，**源码内相对 import 必须写 `.js` 扩展名**（如 `from '../types/m3u8.js'`），tsc 会映射到同名 `.ts`；写成 `.ts` 或无扩展名都会失败——最易踩的坑。

### 下载核心链路（`src/lib/m3u8-download.ts`）

`m3u8Download(url, options)`：

1. `formatOptions` 规范化参数、推导 `filename`/`type`/`urlMd5`、`cacheDir`，按 `isSupportFfmpeg` 决定输出 `.mp4`/`.ts`；目标文件存在且非 `force` 则跳过。
2. `parseM3U8`（基于 `m3u8-parser`）解析，master 列表取 BANDWIDTH 最大子流；`ignoreSegments`（`0-30`、`END-60`）过滤片段。
3. 生成每个 ts 的 `tsOut = <cacheDir>/<md5(url)>/<md5(tsUri)>.ts`，写 `info.json`。
4. `WorkerPool`（`worker_pool.ts`，`worker_threads`，线程数默认 `cpus().length`）投递全部任务；worker 内 `ts-download.ts` 若 `existsSync(tsOut)` 直接返回（**断点续传**），并在此完成 AES-128 解密（key 存于内存 `m3u8Info.crypto[keyUri]`，不落盘）。
5. 回调：失败任务在 `info.success >= -3` 时延迟 1s 重投（最多 3 次）；成功累计 `tsSize`，用近 60s 滑动窗口 `calcSpeed` 算速度与剩余时间，经 `logger.logInline` 和 `options.onProgress` 上报。
6. 全部完成 → `barrier.open()` → `workPoll.close()` → `m3u8Convert`（生成 `ffconcat.txt` 后 `execSync` 调 ffmpeg concat + `-c copy`；无 ffmpeg 则退化为顺序拼接 `.ts`）→ 按 `delCache` 删缓存目录。

`src/m3u8-batch-download.ts` 封装批量/串行：按 `options.type` 分派 `parser`→`VideoParser.download`、`file`→`fileDownload`、`web`→`getM3u8Urls` 嗅探、默认→`m3u8Download`；空闲 worker 多时调 `preDownLoad()` 预取下一集。

边下边播：`lib/local-play.ts` 的 `toLocalM3u8` 用已落盘 ts 拼本地 m3u8，`localPlay` 起本地静态服务（`findFreePort` + `video/mp2t`）并跳转 `https://m3u8-player.lzw.me?url=...`。

缓存布局：`<cacheDir>/<md5(url)>/` 下为 ts 片段、`info.json`、`ffconcat.txt`（成功后删）、`<urlMd5>.m3u8`；`<cacheDir>/` 根下为服务端状态文件。

### 服务端（`src/server/download-server.ts`）

`DLServer` = Express 5 + `ws`，默认端口 6600，缓存目录 `~/.m3u8-dl/cache`。

- REST：`/healthcheck`、`GET|POST /api/config`、`GET /api/tasks`、`GET /api/queue/status`、`POST /api/queue/clear`、`POST /api/download|pause|resume|delete|rename|getM3u8Urls`、`GET /localplay/*`（受 `limitFileAccess` 限制，仅 cacheDir/saveDir）。
- WebSocket：连接即推 `serverInfo`、`tasks`，之后广播 `progress`、`tasks`、`delete`、`queueStatus`；鉴权比对 `?token=`/`authorization` 与 `md5(token).slice(0,8)`，失败 `close(1008)`。
- 持久化：`<cacheDir>/config.json`（`webOptions` + `dlOptions`）与 `cache.json`（`[url, CacheItem][]`，`status: pending|resume|pause|done|error`，载入时 `resume→pause`，防抖 1s 落盘）。并发上限 `webOptions.maxDownloads`。
- 环境变量：`DS_PORT`、`DS_SECRET`、`DS_SAVE_DIR`、`DS_CACHE_DIR`、`DS_FFMPEG_PATH`、`DS_DEBUG`、`DS_PROXY_MODE`、`DS_PROXY_URL`、`DS_NO_PROXY`、`DS_LIMTE_FILE_ACCESS`（`LIMTE` 为原样拼写）。

### CLI（`src/cli.ts`）

commander v12。主命令收多个 url → `m3u8BatchDownload`；子命令 `server`、`search|s`（`lib/video-search.ts` + `lib/search-api/`，enquirer 交互）、`info`（打印 `VideoParser.parse` 结果）。`getOptions()` 统一处理 `--lang`、`--debug/--silent`。

### 视频解析（`src/video-parser/`）

`BaseParser` 提供静态 `success/error` 与默认 `parse(url, headers)`；实现有 `douyin-parser.ts`、`weibo-parser.ts`（需 cookie）、`pipixia-parser.ts`。`VideoParser` 用静态注册表 `platforms`（hostname 去 `www.` 后 `includes` 匹配，兜底 `**`）分发，`download()` 得直链后走 `fileDownload`。

### 前端（`packages/frontend`）

Vite 输出到根 `client/`（`emptyOutDir:false`），dev 代理 `/api`、`/ws`(ws:true)、`/localplay`、`/local`、`/m3u8-capture.user.js` → `http://localhost:6600`。路由 5 页：`/page/download`、`web-browser`、`completed`、`config`、`about`。

Pinia：`config`、`tasks`、`server`、`favorites`（纯 localStorage，未从 `stores/index.ts` 导出）。后端调用集中在 `src/utils/request.ts`（自动带 `authorization: localStorage.token` 与 `lang`）。WebSocket 在 `src/composables/useWebSocket.ts`，**模块级单例** `globalWs`，按 `type` 分发到 store；`1008` 触发 `PasswordDialog`。i18n 语言包 `src/i18n/locales/{zh-CN,en}.ts`，两层 key。

### 前后端类型契约（`src/types/contract.ts`）

服务端与 WebUI 的传输数据结构（`TaskItem`、`TaskStatus`、`TaskOptions`、`DownloadStats`、`QueueStatus`、`ServerInfo`、`ServerConfig`、`ApiResponse`）**只在此文件定义一次**，是唯一真相源：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lzwme/m3u8-dl](https://github.com/lzwme/m3u8-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
