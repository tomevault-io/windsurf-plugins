---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**家庭包厢式 KTV 系统**

这是一个面向家庭客厅场景的单房间包厢式 KTV 系统。电视只负责全屏播放，手机是唯一控制端，系统通过中心服务端统一管理点歌、队列、播放状态和媒体资源。第一版重点不是做“最花哨的 KTV”，而是先把本地歌库为主、在线补歌为辅的稳定可唱体验做扎实。

**Core Value:** 在家庭单电视场景下，让用户用手机完成全部点歌与控制，并稳定地把歌唱起来。

### Constraints

- **Product scope**: 第一版必须优先验证“稳定可唱”而不是“功能最全” — 复杂增强能力要延后
- **Interaction model**: 手机必须是唯一控制端 — 电视端不承载搜索和复杂操作
- **Playback model**: 播放状态只能由服务端状态机裁决 — 防止手机端与电视端状态漂移
- **Audio chain**: 软件不承担实时人声 DSP — 由硬件完成混响、监听、EQ 和最终混音
- **Media source**: 歌曲来源采用“本地为主、在线补歌为辅” — 在线源不允许成为主依赖
- **Online playback**: 第一版在线歌曲必须先缓存再播放 — 提高稳定性并简化失败恢复
- **Room model**: 第一版虽然只有一个房间，但数据模型必须保留 `room` 概念 — 避免后续推倒重来
- **Search quality**: 中文搜索体验必须覆盖歌名、歌手、拼音、首字母、别名与繁简体 — 否则点歌体验不可接受
- **Deployment**: 预期运行在家庭服务器拓扑中，业务与任务在 `lxc-dev`，歌库与缓存位于 `lxc-nas`
- **Compliance**: 在线歌源接入需要遵守明确的合规边界 — 具体 provider 选择与策略要在实施前再确认
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Node.js | 24.15.0 LTS | Backend runtime for API and worker processes | Current LTS line with mature ecosystem support; a good fit for one TypeScript codebase spanning API, worker, and tooling. |
| TypeScript | 5.9.2 | Shared type system across TV, mobile, API, worker, and domain packages | This project’s biggest long-term risk is contract drift across multiple runtimes. TypeScript reduces that risk materially. |
| React | 19.2.5 | UI layer for `tv-player`, `mobile-controller`, and lightweight admin | Strong ecosystem, mature browser support, and easy reuse of design/system logic across the three frontends. |
| Vite | 8.0.10 | Frontend build/dev tool for TV, mobile, and admin apps | Fast local iteration, straightforward multi-app setup, and good fit for browser-first deployment. |
| Fastify | 5.8.5 | HTTP + WebSocket server for commands, queries, pairing, and realtime fanout | Lightweight, performant, schema-friendly, and easier to keep deterministic than heavier opinionated frameworks for this stateful product. |
| PostgreSQL | 18.3 | Durable store for catalog metadata, queue history, source records, pairing tokens, and playback events | Strong relational modeling, JSONB where needed, and `pg_trgm` support for incremental Chinese search/read-model work before introducing a separate search engine. |
| NAS + FFmpeg/ffprobe | Current stable line | Controlled media storage plus probe/validation tooling | Phase 1 depends on stable local assets and media inspection more directly than it depends on a background job stack. |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Zod | 4.3.6 | Runtime validation for API schemas, command payloads, and event envelopes | Use at all external boundaries: HTTP input, WebSocket messages, config, and worker job payloads. |
| Drizzle ORM | 0.45.2 | Typed SQL access for the catalog/session database | Use if the team wants SQL-first control and shared TS types without hiding schema design behind a heavy abstraction. |
| BullMQ | 5.76.2 | Queueing for library scans, online-cache jobs, verification, and retries | Introduce only after Phase 1 if async asset-readiness workflows outgrow a single backend process. |
| TanStack Query | 5.100.5 | Server-state caching in the mobile controller and admin UI | Use where the UI consumes snapshots/lists from the API but still needs background refresh and optimistic affordances. |
| TanStack Router | 1.168.25 | Route/state composition for mobile and admin frontends | Use if the apps grow beyond a single screen and need typed route params plus cleaner route-level data loading. |
| Zustand | 5.0.12 | Lightweight local UI state for player controls, room presence, and transient interaction state | Use for local UI state that should not be mixed into the server-authoritative room/session model. |
| `pg` | 8.x stable line | PostgreSQL driver beneath Drizzle and direct SQL tooling | Use for database connectivity and maintenance scripts. |
| `ioredis` | 5.x stable line | Redis client for room cache, pub/sub, and BullMQ | Add only if later phases prove Redis is needed for hot-state or queue orchestration. |
| `@fastify/websocket` | 11.x stable line | WebSocket support for session snapshots and player telemetry | Use when Fastify handles both REST and realtime transport in one process. |
| `hls.js` | 1.x stable line | HLS fallback for future streaming-compatible assets | Use only if a later phase introduces HLS playback; do not pull it into MVP if `mp4`/static assets cover the target devices. |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| pnpm 10.33.2 | Monorepo package manager | Good workspace ergonomics, fast installs, and sensible lockfile behavior for multi-app TypeScript repos. |
| Turborepo 2.9.6 | Task orchestration across apps/packages | Useful once API, worker, TV, mobile, and admin share builds, checks, and generated artifacts. |
| Caddy 2.11.2 | Reverse proxy and static asset/media serving | Clean fit for home/self-hosted deployment, TLS termination, and exposing controlled media routes. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShaoLongFei/home-ktv-system](https://github.com/ShaoLongFei/home-ktv-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
