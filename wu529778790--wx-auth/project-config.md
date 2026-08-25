---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

这是一个**微信订阅号认证系统**，核心功能是通过微信扫码关注公众号获取验证码，输入6位验证码完成认证。认证成功后保存状态，下次访问自动认证（无需重复操作）。

**技术栈：**
- **前端**: Nuxt 4 + Vue 3 + TypeScript + Tailwind CSS
- **后端**: Nitro Server API (Node.js)
- **SDK**: 原生 JavaScript + Vite 构建（< 12KB，零依赖）
- **存储**: Turso（libSQL 托管）单一后端，对外统一 async 接口

## 快速导航

- **后端 API**: `server/api/` - Nitro Server API endpoints
- **工具函数**: `server/utils/` - 微信加解密、存储、Session 管理
- **SDK 源码**: `wx-auth-sdk/src/` - 前端 SDK 核心逻辑
- **前端页面**: `pages/` - Nuxt 页面路由
- **数据目录**: `data/` - 本地 Turso 文件库（TURSO_URL=file: 时）及历史备份

## 开发命令

### 主项目
```bash
npm install          # 安装依赖
npm run dev          # 开发模式（http://localhost:3000）
npm run build        # 构建生产版本
npm run preview      # 预览生产构建
npm run generate     # 生成静态站点
```

### SDK 开发
```bash
cd wx-auth-sdk
npm install          # 安装依赖
npm run dev          # SDK 开发模式
npm run build        # 构建 SDK（输出到 dist/）
npm run type-check   # TypeScript 类型检查
```

### Docker 部署
```bash
# 本地构建测试
docker build -t wx-auth .
docker run -d --name wx-auth --env-file .env -p 3000:3000 -v ./data:/app/data wx-auth

# 查看日志
docker logs -f wx-auth

# 停止容器
docker stop wx-auth && docker rm wx-auth
```

## 核心架构

### 后端 API 端点
1. **`/api/wechat/message`** - 微信消息处理（GET/POST）- 接收微信服务器推送
2. **`/api/auth/check`** - 认证检查（参数：`authToken` 或 `openid`，可选 `siteId`）
3. **`/api/auth/userinfo`** - 用户详情（业务授权用：`role` / `vipExpiresAt` / `isAdmin`，参数 `token` 或 Cookie；不提供明文 openid 通道）
4. **`/api/auth/session`** - Session 管理（POST/GET/DELETE）
5. **`/api/sdk/config`** - SDK 配置下发（返回 wechatName、qrcodeUrl）

### 后台管理 API（需管理员登录，身份 = 用户表 `is_admin` 标记）
1. **`POST /api/admin/login`** - 管理员登录（接收微信签名 Token，校验 `is_admin`，写 `wxauth-admin` Cookie；免密）
2. **`POST /api/admin/logout`** - 管理员登出
3. **`GET /api/admin/stats`** - 统计看板（累计/有效/今日/流失率，按站点分组）
4. **`GET /api/admin/users`** - 用户列表（分页 + siteId/status/role 筛选 + keyword 搜索）
5. **`PATCH /api/admin/users/[openid]`** - 编辑用户（role / vipExpiresAt / remark / status / isAdmin）

> 鉴权：`server/middleware/admin-guard.ts` 守卫所有 `/api/admin/*`（login 除外），未登录返回 401。登录态基于加密 Cookie（AES-256-GCM），与用户认证 Session 隔离。

### 前端页面
- **`pages/index.vue`** - 认证演示页面 + SDK 接入文档（合并展示）
- **`pages/admin.vue`** - 后台管理（数据看板 + 用户列表，Tabs 切换）
- **`pages/admin/login.vue`** - 后台登录页（微信认证免密登录）

### SDK 工作流程
1. 检查 Cookie `wxauth-openid`
2. 已认证 → 静默通过
3. 未认证 → 显示弹窗（二维码 + 6位输入框）
4. 用户扫码 + 输入验证码
5. 验证成功 → 保存 Cookie + 回调
6. 用户关闭弹窗 → 触发 `onClose` 回调

### 工具层（server/utils/）
- **`wechat.ts`** - 微信 API 交互（签名验证、消息解析、加密解密）
- **`storage.ts`** - 存储层抽象（Turso 单一后端，按 siteId 隔离）
- **`turso.ts`** - Turso 访问层（@libsql/client，惰性 prepare + 事务 batch + 幂等建表）
- **`session.ts`** - Session 生成与验证（AES-256-GCM 加密）
- **`token.ts`** - Token 生成与管理
- **`rate-limit.ts`** - 速率限制（防止暴力破解）
- **`admin-auth.ts`** - 后台管理员登录态（独立 Cookie，AES-256-GCM 加密）

### 用户数据模型（storage.ts）
```ts
interface AuthenticatedUserData {
  authenticatedAt: string;
  nickname?: string;
  headimgurl?: string;
  unionid?: string;
  siteId?: string;
  status?: 'active' | 'unsubscribed' | 'disabled';  // active=有效 / unsubscribed=已取关 / disabled=被封禁
  unsubscribedAt?: string;   // 取关时间（软删除留痕）
  role?: 'normal' | 'vip';   // 角色
  vipExpiresAt?: number;     // VIP 到期时间戳（毫秒），null 表示永久
  remark?: string;           // 管理员备注
}
```

> **软删除约定**：取关事件 `clearUserAuthentication` 只打 `status='unsubscribed'` 标记并记录时间，**不物理删除**。认证检查（`isUserAuthenticated` / `getAuthenticatedUser`）仅认 `active`（老数据无 status 视为 active），重新认证自动复活为 `active` 且保留 role/remark。后台统计区分「累计（totalAuth）」与「有效（totalActive）」。

## 关键配置

### 环境变量（.env）
```bash
# 必须（Docker 部署必须加 NUXT_ 前缀，否则构建时被内联为空值）
NUXT_PUBLIC_SITE_URL=https://wx-auth.shenzjd.com
NUXT_WECHAT_TOKEN=your-wechat-token
NUXT_SESSION_SECRET=dev-secret-change-in-production
# 后台已改为免密登录（is_admin 角色判定），无需 ADMIN_PASSWORD；
# 首次设置管理员用：npx tsx scripts/set-admin.ts <openid>
NUXT_WECHAT_NAME=神族九帝
NUXT_WECHAT_QRCODE_URL=https://your-site.com/qrcode.jpg

# 可选（个人订阅号留空）
NUXT_WECHAT_AES_KEY=
NUXT_CODE_EXPIRY=300
NUXT_KEYWORDS=["验证码"]      # 触发关键词，支持简繁体（发"驗證碼"也可触发）
NUXT_WECHAT_MENU_KEY=GET_CODE  # "验证码"菜单的事件 key（需与公众号后台配置一致）
# 存储后端：turso（唯一，默认；设置其他值会启动报错退出，防数据分叉）
# 注意：STORAGE_TYPE 无 NUXT_ 前缀，直接读环境变量，Docker 部署时通过 -e STORAGE_TYPE=turso 传入
# Turso 模式需配置 TURSO_URL / TURSO_TOKEN（Turso 控制台 Database → Generate Token）
STORAGE_TYPE=turso
```

### SDK 配置
```typescript
// ✅ 零配置接入（推荐）
WxAuth.init({
  onVerified: (user) => { ... },  // ← 唯一必填：验证成功回调
  onClose: () => { ... }          // ← 可选：关闭弹窗回调
});

// 或手动指定配置（可选）
WxAuth.init({
  apiBase: 'https://wx-auth.shenzjd.com',  // 后端 API 地址（可选，有默认值）
  required: false,                     // 是否必须认证（默认 true）
  silent: false,                       // ← 可选：true 时 init 不调弹窗，弹窗由 requireAuth() 手动触发
  // wechatName 和 qrcodeUrl 无需配置，自动从后端获取
  // 接入方配置自己的公众号名称和二维码无效（统一使用"神族九帝"）
  onVerified: (user) => { ... },       // 验证成功回调
  onError: (error) => { ... },        // 错误回调
  onClose: () => { ... }               // 关闭弹窗回调（仅 required=false 时触发）
});

// ✅ silent 模式：延迟弹窗（例：免费搜索 3 次后再要求认证）
WxAuth.init({
  silent: true,
  required: false,
  onVerified: (user) => { /* 标注已认证 */ },
});
// 业务代码里：freeSearches 用完后 await WxAuth.requireAuth() 手动触发弹窗
```

**核心概念：**
- ✅ **所有参数都无需手动配置**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wu529778790/wx-auth](https://github.com/wu529778790/wx-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
