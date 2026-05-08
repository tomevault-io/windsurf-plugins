---
trigger: always_on
description: Libra AI Project Structure Guidelines - Comprehensive Monorepo Architecture Rules and Best Practices
---


# Libra AI Project Structure Guidelines

## Overview

Libra AI is a modern open-source AI-driven development platform built on **Turborepo Monorepo** architecture. This document defines the project structure rules, naming conventions, and organizational patterns that must be followed when working with the codebase.

## Core Architecture Principles

### 1. Monorepo Organization
- **Turborepo First**: All code is organized in a unified monorepo with clear separation between applications and shared packages
- **Separation of Concerns**: Clear boundaries between application layer, business logic layer, and data layer
- **Type Safety First**: End-to-end TypeScript type coverage with Zod validation
- **Developer Experience First**: Bun package manager, Biome code formatting, hot reload support

### 2. Directory Structure Rules

```text
libra/
├── apps/                    # Applications (独立可部署的应用)
│   ├── auth-studio/         # 认证管理控制台 (D1 + drizzle-kit)
│   ├── builder/             # Vite 构建服务 - 代码编译与部署
│   ├── cdn/                 # Hono CDN 服务 - 静态资源管理
│   ├── deploy/              # 部署服务 V2 - Cloudflare Queues
│   ├── deploy-workflow/     # 部署服务 V1 - Cloudflare Workflows (deprecated)
│   ├── dispatcher/          # 请求路由分发器 (Workers for Platforms)
│   ├── docs/                # 技术文档站点 (Next.js + FumaDocs)
│   ├── email/               # 邮件服务预览器 (React Email)
│   ├── screenshot/          # 截图服务 - Cloudflare Queues
│   ├── vite-shadcn-template/# 项目模板引擎 (Vite + shadcn/ui)
│   └── web/                 # Next.js 15 主应用 (React 19)
├── packages/                # 共享包模块
│   ├── api/                 # API 层 (tRPC + 类型安全)
│   ├── auth/                # 认证服务 (better-auth)
│   ├── better-auth-cloudflare/ # Cloudflare 认证适配器
│   ├── better-auth-stripe/  # Stripe 支付集成
│   ├── common/              # 公共工具库与类型定义
│   ├── email/               # 邮件服务组件
│   ├── middleware/          # 中间件服务与工具
│   ├── sandbox/             # 统一沙箱抽象层 (E2B + Daytona)
│   ├── shikicode/           # 代码编辑器 (Shiki 语法高亮)
│   ├── templates/           # 项目脚手架模板
│   └── ui/                  # 设计系统 (shadcn/ui + Tailwind CSS v4)
├── tooling/                 # 开发工具和配置
│   └── typescript-config/   # 共享 TypeScript 配置
├── scripts/                 # Github 环境变量管理
├── biome.json               # Biome 配置
├── bun.lockb                # Bun 锁定文件
├── package.json             # 根级依赖管理 (Bun workspace)
└── turbo.json               # Turborepo 构建配置
```

## Application Structure Rules

### apps/web - Main Web Application
**Technology**: Next.js 15 + React 19 + App Router

```text
apps/web/
├── ai/                      # AI integration and providers
│   ├── models.ts            # AI model configuration and selection logic
│   ├── generate.ts          # Core logic for code generation
│   └── prompts/             # AI prompt template directory
├── app/                     # Next.js App Router
│   ├── (frontend)/          # Frontend route group
│   │   ├── (dashboard)/     # Dashboard page group
│   │   │   ├── dashboard/   # User dashboard
│   │   │   └── project/     # Project management page
│   │   └── (marketing)/     # Marketing page group
│   └── api/                 # API routes
│       ├── ai/              # AI-related APIs
│       ├── auth/            # Authentication APIs
│       ├── trpc/            # tRPC endpoints
│       └── webhooks/        # Webhook handling
├── components/              # React components
│   ├── auth/                # Authentication components
│   ├── billing/             # Billing and subscription components
│   ├── dashboard/           # Dashboard components
│   ├── ide/                 # IDE editor components
│   ├── marketing/           # Marketing page components
│   └── ui/                  # Base UI components
├── hooks/                   # Custom React Hooks
├── lib/                     # Utility function library
├── trpc/                    # tRPC client configuration
└── env.mjs                  # Environment variable configuration
```

**Rules for apps/web**:
- All pages must use App Router structure
- Components should be organized by feature/domain
- Custom hooks must be in `/hooks` directory
- Utility functions go in `/lib` directory
- AI-related code must be in `/ai` directory

### Other Applications Structure

#### apps/builder - Vite Build Service
```text
apps/builder/
├── src/
│   ├── components/          # Build tool UI components
│   ├── lib/                 # Build logic and utilities
│   └── utils/               # Helper functions
├── vite.config.ts           # Vite configuration
└── wrangler.jsonc           # Cloudflare Workers configuration
```

#### apps/cdn - CDN Service (Hono)
```text
apps/cdn/
├── src/
│   ├── routes/              # API route handlers
│   ├── middleware/          # Request middleware
│   └── utils/               # Utility functions
├── wrangler.jsonc           # Workers configuration
└── package.json
```

#### apps/dispatcher - Request Router (Hono)
```text
apps/dispatcher/
├── src/
│   ├── middleware/          # Authentication middleware
│   ├── routes/              # Routing logic
│   └── utils/               # Utility functions
├── wrangler.jsonc           # Cloudflare Workers configuration
└── package.json
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextify-limited/libra](https://github.com/nextify-limited/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
