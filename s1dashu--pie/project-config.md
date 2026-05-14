---
trigger: always_on
description: AGENTS.md 是给 coding agents 的项目工作指南。先读本文件，再按需阅读 [DESIGN.md](./DESIGN.md)、源码和相邻文档。
---

# pie

AGENTS.md 是给 coding agents 的项目工作指南。先读本文件，再按需阅读 [DESIGN.md](./DESIGN.md)、源码和相邻文档。

## Project Overview

Pie 是一个个人 Agent 客户端产品，不是单纯的 coding bot。Pie 是客户端和产品名，不是 agent framework 名。

当前 runtime 主要基于 `@mariozechner/pi-coding-agent` 的 session/tool 能力。产品侧可以选择不同 harness/backend：默认稳定路径是 Pi；Ousia 是显式选择后才启用的独立 framework companion；Codex、Hermes、OpenClaw 都是 Pie 侧 adapter 接入的本机官方 runtime，其中 Hermes 默认按 profile 拉起本地 gateway，OpenClaw 默认复用共享官方 gateway/service，启动资源成本明显高于 Pi/Ousia session。

核心原则：

- 保持架构小而清楚，避免为了未来可能性过早抽象。
- 先做好客户端产品体验和长期工作能力，再扩展复杂记忆、知识或插件系统。
- 当前产品未上线，没有历史用户包袱；不要保留或恢复旧 `momo`、旧 config、旧 profile 兼容层。
- 保持文档描述当前真实状态，不要把规划、原型或早期接入写成稳定产品能力。

## Current Product State

- 根目录 `pie` 是客户端产品 runtime：desktop、CLI/onboard、channel adapters、profile/config/agent home 管理。
- 当前主要开发对象是桌面端；CLI/onboard、channel adapters 等能力服务于桌面端 Agent 客户端体验。
- 当前支持的 IM channel 是 `feishu`、`wechat`、`discord` 和 `dingtalk`；其中 `feishu` 完成度较高，`wechat` 已有扫码登录、轮询和收发消息实现，`discord` 已接入桌面创建入口和 runtime，`dingtalk` 已接入应用机器人 Stream 模式文本收发。
- `slack`、`telegram` 仍在支持中，桌面开发者模式开启后才开放，本地 runtime 在非开发者模式下会跳过它们。
- Pi / `pi-coding-agent` 是当前真正稳定运行的 harness/backend，默认创建新 Agent 时选择 Pi。
- Ousia 复用 `pi-coding-agent` session，但有自己的 system prompt、tools 配置、Task Engine 和 `/agent/run` gateway。
- Codex 通过本机官方 Codex CLI 的 `app-server --listen stdio://` 接入；桌面有安装/登录诊断和模型选择，但 permission/plan approval 的 IM 交互仍未完成。
- Hermes 复用本机官方 Hermes runtime/service；Pie 可负责发现、启动、健康检查和 adapter 接入，但不应替代 Hermes 自己的 profile/runtime 语义。当前 Hermes 多 profile 按多个 Hermes home 建模：一个 Pie profile 默认对应 `<profile-home>/hermes` 和一个 profile-local gateway/API server port，不走 OpenClaw 式共享 gateway。桌面创建流程会做本机诊断/安装；仍按早期接入处理。
- OpenClaw 是真实接入。Pie 复用本机官方 `openclaw`、官方 `~/.openclaw/openclaw.json` 和官方 gateway/multi-agent 能力；桌面端优先由 `SharedHarnessServiceRegistry` 管理同一个官方 OpenClaw gateway，再把 Pie profile 映射为 namespaced OpenClaw agent。OpenClaw 资源成本高，桌面自动恢复时要受启动预算、延迟调度和资源余量约束。Pie 不内置 Node，不静默安装 OpenClaw；本机缺失或不可用时只通过桌面 UI 给用户显式“安装官方版 OpenClaw”的入口。
- Task Engine 的 scheduled task、runtime heartbeat 和 Ousia daily session distillation 仍偏原型；不要描述为稳定产品能力。

## Development Commands

- 安装依赖：`npm install`
- 类型检查：`npm run check`
- 测试：`npm run test`
- Live/profile 测试分层和矩阵说明见 [docs/testing.md](./docs/testing.md)；完整 live matrix 用 `npm run test:live`
- 启动 runtime：`npm run start` 或 `pie`
- 构建：`npm run build`
- Desktop 开发：`npm run desktop:dev`
- Desktop 构建：`npm run desktop:build`（只生成 `out/`，不生成 `Pie.app`）
- Desktop 打包 Pie.app：`npm run desktop:build && CSC_IDENTITY_AUTO_DISCOVERY=false npx electron-builder --mac dir && open dist/mac-arm64/Pie.app`，产物通常在 `dist/mac-arm64/Pie.app`。默认只打包 `Pie.app`；除非用户明确要求打包 DMG，不要跑 `electron-builder --mac dmg`。
- Desktop 打包 DMG/ZIP：`npm run desktop:build && npx electron-builder --mac dmg zip --publish never`。只在用户明确要求 DMG 时使用。

修改代码后至少跑 `npm run check`；改到已覆盖测试的模块时也跑 `npm run test` 或相关 `tsx --test` 子集。涉及构建入口、desktop、runtime 时也跑对应 build。

## Design Instructions

Desktop UI、视觉 tokens、组件样式和交互规范统一维护在 [DESIGN.md](./DESIGN.md)。不要在 AGENTS.md 里重复 UI/design 规则。

## Official Website / Landing Page

Pie 官网是相邻独立 repo `/Users/bytedance/code/pie-landing-page`，不是本 repo 的 workspace package。用户提到官网、landing page、下载页、SEO、非登录态首页或公开产品介绍时，优先切到该 repo，并先读它自己的 `AGENTS.md`。

官网维护规则：

- `pie-landing-page` 是 Next.js app，基于 SaaS landing page template；尽量保留模板视觉风格，除非用户明确要求改视觉资产。
- 官网内容以本 repo 的 `README.md` 和 `AGENTS.md` 为产品事实源；文案要营销友好但保守，不要把原型、早期接入或未稳定能力写成已稳定发布。
- 当前官网已有英文根路径 `/` 和中文路径 `/zh`；如要做地区或语言默认跳转，应在 `pie-landing-page` 中实现并验证，不要在 Pie desktop/runtime 里处理。
- 当前公开下载主要面向 macOS；不要添加 Windows/Linux 下载按钮，除非 Pie 实际已经发布对应构建。
- DMG 体积过大，不能放入官网 `public/`；下载链接使用 Vercel Blob URL，更新安装包时按 `pie-landing-page/AGENTS.md` 的 Vercel Blob 流程处理。
- 官网常见同步文件包括 `src/app/layout.tsx`、`src/app/page.tsx`、`src/app/zh/page.tsx`、`src/components/Hero.tsx`、`src/components/PrimaryFeatures.tsx`、`src/components/Faqs.tsx`、`src/components/Testimonials.tsx` 和官网 `README.md`。
- 修改官网后在 `/Users/bytedance/code/pie-landing-page` 运行 `npm run lint` 和 `npm run build`。用户要求更新线上站点时，通常 commit + push `origin/main` 触发 Vercel Git 部署；不要默认跑 `npx vercel --prod`，除非用户明确要求或 Git 部署不可用。

## Architecture Rules

- 一个 profile = 一个 agent instance。
- 一个 agent instance 的 config 可以挂多个 channels。当前多 channel 主要是配置结构和启动能力预留，还没有完整的一等路由策略。
- 未来同一个 bot 同时接 Feishu/Wechat/Slack 时，应在同一个 profile 下挂多个 channel adapter，而不是多个独立 bot。
- `config.json` 使用当前结构：`profile.harness + profile.channels[]`。旧 `profile.backend` 只作为读入兼容 fallback，不要写回或新增。
- 敏感信息只进 `<agent-home>/.env`，不要写入 `config.json`。
- Pie instance 编排入口放在 `src/runtime/`；agent/framework 层能力不要直接放在这里。
- `channel` 目录只负责 channel adapter：收消息、发消息、channel 事件解析与投递。不要让 channel 负责启动 Task Engine、run gateway 或 harness managed service。
- Runtime 选择和创建 enabled channel 的入口集中在 `src/runtime/channel-runtimes.ts`，release/developerMode 可见性由 `src/core/channel-availability.ts` 控制。
- Harness/backend 抽象保持轻量，注册入口集中在 `src/agents/harness-registry.ts`。不要再分别维护多套 capability 表。
- 外部 framework/backend 源码不要改；协议差异在 Pie 自己的 adapter / normalizer 中处理。
- Codex、Hermes、OpenClaw 这类外部 harness 的 runtime ownership 归各自官方 runtime；Pie 只做 discovery、managed start/stop、健康检查、profile 映射、会话 adapter 和 IM 编排，不 fork、不 patch、不重建一套 runtime-level multi-profile。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s1dashu/pie](https://github.com/s1dashu/pie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
