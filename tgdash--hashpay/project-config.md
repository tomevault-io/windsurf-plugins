---
trigger: always_on
description: 本文件帮助快速理解 HashPay 的项目结构、运行原理和开发约定。
---

# AGENTS.md

本文件帮助快速理解 HashPay 的项目结构、运行原理和开发约定。
修改代码需严格遵守。

## 项目概述

HashPay 是一个运行在 Cloudflare Workers 环境上的加密货币收款系统，项目同时包含后端 Worker 服务和 Vue 管理/收银台前端。

主要能力包括：

- 管理员后台：配置商户、收款通道、订单、系统设置、Banner 和汇率微调。
- 商户 API：通过 RSA 签名创建订单、查询订单，并返回托管收银台链接。
- 收银台：选择资产和网络、展示地址/二维码、触发服务端查账、超时和支付成功状态处理。
- Telegram 支持：绑定管理员、Mini App 后台入口、PIN 登录、Bot 内创建收款订单、获得订单通知。
- 支付检测：支持自动链上检查、交易所/钱包 API 查账、用户触发服务端查账和人工确认。
- 回调通知：订单支付成功后发送回调消息，利用队列系统确保投递稳定。
- 汇率管理：定时同步法币兑 USD 汇率，USD / USDT / USDC 按 1:1 处理，并支持后台微调。

## 技术栈

- 语言：TypeScript，ESM，严格类型检查。
- 前端：Vue 3 + Vue Router + Naive UI + Vite + SCSS。
- 后端：Cloudflare Workers + Hono。
- 数据库：Cloudflare D1，SQL 迁移文件位于 `src/server/db/d1/migrations/`。
- Cloudflare：Workers Assets、D1、Queues、Cron Triggers。
- Telegram：grammY + Telegram Bot API。
- 测试：Vitest。

## 常用命令

```bash
npm install
npm run dev
npm run dev:worker
npm run check
npm run test
npm run build
npm run deploy:dry
npm run cf-typegen
```

- `npm run dev` 启动 Vite + Cloudflare Vite 插件，本地前端端口见 `vite.config.ts`。
- `npm run dev:worker` / `npm start` 直接启动 `wrangler dev`，端口见 `wrangler.jsonc`。
- `npm run db:migrate:local` / `npm run db:migrate:remote` 分别应用本地/远程 D1 迁移。
- 修改 Cloudflare bindings 后运行 `npm run cf-typegen`，并同步更新 `src/server/types/env.ts`。
- 不要提交 `.dev.vars`、真实 Bot Token、APP_SECRET、商户私钥或其他密钥。

## 目录结构

```text
HashPay/
├── src/index.ts                         # Worker 入口：fetch、scheduled、queue
├── src/server/http/                     # Hono 应用、中间件、API envelope、路由
│   └── routes/                          # public、auth、admin 路由
├── src/server/services/                 # 后端业务服务
│   ├── app/                             # 系统状态、设置、汇率、定时任务
│   ├── auth/                            # Session、Telegram initData、PIN 登录
│   ├── images/                          # Banner 和订单二维码
│   ├── merchants/                       # 商户、RSA 密钥、商户签名校验
│   ├── orders/                          # 建单、收银台、订单管理、通知
│   └── telegram/                        # Bot API、Webhook、初始化绑定
├── src/server/payments/                 # 支付通道模型、快照生成、查账驱动
│   └── providers/                       # 付款渠道的查询实现
├── src/server/db/                       # D1 helper、配置 helper、迁移执行器
├── src/server/types/                    # Worker 环境类型和 Hono 变量
├── src/shared/                          # 前后端共享类型
├── src/app/                             # Vue 前端应用
│   ├── api/                             # 前端 API client，复用 shared/types/api.ts
│   ├── components/                      # 前端组件
│   ├── pages/                           # 前端页面
│   ├── payments/                        # 前端支付展示和浏览器侧查账能力
│   ├── utils/                           # 前端格式化、剪贴板、收银台状态机等工具
│   ├── i18n.ts                          # 前端轻量 i18n store
│   └── styles.scss                      # 全局样式
├── test/                                # Vitest 单元测试
├── wrangler.jsonc                       # Cloudflare Workers 配置
├── vite.config.ts                       # Vite + Cloudflare 插件配置
└── vitest.config.ts                     # Vitest 配置
```

## Worker 运行方式

- `src/index.ts` 是唯一 Worker 入口，导出 `fetch`、`scheduled`、`queue`。
- `fetch` 进入 `createApp()`，Hono 在 `src/server/http/app.ts` 装配中间件、路由和 Assets 回退。
- API 成功响应直接返回业务 JSON，不包 `{ data }`；错误统一走 `{ error: { key, params } }`，前端负责按当前语言翻译。
- `/api/*` 请求会先执行 `migrateD1(c.env)`，但 `/api/state` 例外；`appState()` 自行尝试迁移并把 DB 错误转成状态字段。
- 未命中后端路由时回落到 `ASSETS.fetch()`，用于服务 SPA。
- Cron 每分钟执行 `runJobs()`：迁移、整点同步汇率、过期订单、检查待支付订单、把到期通知放入队列。
- Queue 消费 `QUEUE_NOTIFY`，消息体需要包含 `notifyId`，失败时延迟重试。

## Cloudflare 绑定

当前 `wrangler.jsonc` 绑定：

- `ASSETS`：Workers Assets，目录 `dist`，SPA fallback。
- `DB`：D1 数据库，迁移目录 `src/server/db/d1/migrations`。
- `QUEUE_NOTIFY`：通知投递队列。
- Cron：`* * * * *`。

环境变量：

- `APP_SECRET`：JWT Session、PIN 登录挑战签名等服务端签名用途。
- `TGBOT_TOKEN`：Telegram Bot API。

改动绑定或环境变量时必须同步检查：

- `wrangler.jsonc`
- `src/server/types/env.ts`
- 依赖这些字段的服务和测试
- `npm run cf-typegen` 生成结果

## 数据模型

D1 初始 schema 在 `src/server/db/d1/migrations/0001_init.sql`：

- `configs`：系统配置、Bot 状态、管理员、Banner blob、汇率缓存等。
- `merchants`：商户、公钥、回调 URL、状态和类型。
- `payments`：收款通道、driver、地址、支持资产、凭据和状态。
- `orders`：订单主体，`merchant_no` 是商户侧幂等单号，`payment` 是支付快照 JSON。
- `notify`：商户回调任务、重试状态、payload 和错误信息。
- `review`：用户提交的人工审核答案、截图和审核状态。
- `d1_migrations`：由迁移执行器自动创建，作为迁移是否已应用的事实来源。

数据库字段使用 snake_case；服务和 API DTO 使用 camelCase。`orders.merchant_no` 对应公开字段 `merchantNo`，不要重新引入旧字段名或兼容分支，除非需求明确要求接入旧外部协议。

## 核心链路

### 初始化

1. 前端 `/setup` 调用 `/api/state` 检查 DB、Queue、Bot。
2. 环境就绪后，前端提交公网 HTTPS 域名到 `/api/admin/setup`。
3. `startTelegramSetup()` 写入 domain、bot_secret，刷新 Bot 信息并设置 webhook。
4. 管理员访问 Bot 并发送任意消息，`bindSetupAdmin()` 写入默认设置、默认 Banner、Mini App 菜单，同步一次汇率，然后写入 `admin_id` 和 `admin_user`。
5. `admin_id` 是实例是否已安装的唯一指示；写入后再次提交 `/api/admin/setup` 会直接返回已初始化错误。
6. 安装完成后进入后台走 Telegram Mini App 登录或浏览器 PIN 登录，不再通过 `/api/admin/setup` 签发 Session Cookie。

初始化完成是内部状态变化，不要新增公开 finalize/status 流程，除非需求明确要求。

### 登录

- Telegram Mini App 登录走 `/api/admin/session/telegram`，使用 `validateWebAppInitData()` 校验 initData。
- 浏览器 PIN 登录走 `/api/admin/session/pin` 创建 challenge，再由管理员在 Bot 中发送 `/login <pin>` 确认。
- Session Cookie 名称为 `hashpay_session`，JWT 由 `APP_SECRET` 签名，有效期 7 天。
- Admin 路由在 setup/session 之后统一 `requireAdmin()` 保护。

### 商户 API 和建单

1. 商户请求 `/api/merchant/new`，使用 `X-Merchant-Id`、`X-Signature`、`X-Timestamp`。
2. 签名原文是 `METHOD\npathname+search\ntimestamp\nbody`。
3. `requireSignedMerchant()` 校验时间窗口、公钥、商户状态和 RSA-SHA256 签名。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TGDash/HashPay](https://github.com/TGDash/HashPay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
