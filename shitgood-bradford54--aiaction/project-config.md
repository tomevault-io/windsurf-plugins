---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a NestJS backend scaffold using Prisma (PostgreSQL) and Redis for caching. The application follows a modular architecture with global services for database and cache access.

## Essential Commands

### DX CLI - 统一管理工具 (推荐)

项目提供了统一的 `dx` CLI 工具用于管理所有开发任务:

```bash
# 启动服务
./scripts/dx start dev          # 启动开发服务器 (默认)
./scripts/dx start debug        # 启动调试服务器
./scripts/dx start prod         # 启动生产服务器
npm run dx start dev            # 或通过 npm 运行

# 构建应用
./scripts/dx build              # 构建应用 (默认开发环境)
./scripts/dx build --prod       # 构建生产版本

# 数据库操作
./scripts/dx db generate        # 生成 Prisma Client
./scripts/dx db migrate --dev   # 执行数据库迁移 (开发环境)
./scripts/dx db migrate --prod  # 执行数据库迁移 (生产环境)
./scripts/dx db reset --dev -Y  # 重置数据库 (跳过确认)
./scripts/dx db studio          # 打开 Prisma Studio
./scripts/dx db seed --dev      # 执行数据库种子

# 测试
./scripts/dx test unit          # 运行单元测试
./scripts/dx test e2e           # 运行 E2E 测试
./scripts/dx test cov           # 运行测试并生成覆盖率报告
./scripts/dx test watch         # 监视模式运行测试

# 代码质量
./scripts/dx lint               # 代码检查
./scripts/dx format             # 代码格式化

# 环境管理
./scripts/dx env setup --dev    # 设置开发环境
./scripts/dx env setup --prod   # 设置生产环境
./scripts/dx env validate       # 验证环境变量

# 清理操作
./scripts/dx clean dist         # 清理构建产物
./scripts/dx clean deps         # 重新安装依赖
./scripts/dx clean all -Y       # 清理所有 (跳过确认)
```

**环境标志说明:**
- `--dev` 或 `--development` - 开发环境 (默认)
- `--prod` 或 `--production` - 生产环境
- `--test` - 测试环境
- `--e2e` - E2E 测试环境
- `-Y` 或 `--yes` - 跳过所有确认提示
- `-v` 或 `--verbose` - 详细输出
- `-h` 或 `--help` - 显示帮助信息

**自动确认:**
在 CI 环境中自动跳过确认:
- 设置 `CI=true`
- 或设置 `AI_CLI_YES=1` / `YES=1`

### 传统命令 (仍然可用)

如果不使用 dx CLI，以下命令仍然可用:

**Development:**
```bash
npm run start:dev          # Start with hot-reload (NODE_ENV=development)
npm run start:debug        # Start with debugging enabled
```

**Environment Setup:**
```bash
npm run env:setup          # Initialize environment files (default: development)
npm run env:setup:dev      # Setup development environment
npm run env:setup:prod     # Setup production environment
npm run env:setup:test     # Setup test environment
npm run env:setup:e2e      # Setup E2E test environment
npm run env:validate       # Validate environment variables
```

**Build & Production:**
```bash
npm run build              # Compile TypeScript to dist/
npm run start:prod         # Run compiled application
```

**Database (Prisma):**
```bash
npm run prisma:generate    # Generate Prisma Client (required after schema changes)
npm run prisma:migrate     # Create and apply migration
npx prisma migrate dev --name <migration-name>  # Named migration
npm run prisma:studio      # Open Prisma Studio GUI
```

**Testing:**
```bash
npm run test               # Run all tests
npm run test:watch         # Watch mode
npm run test:cov           # With coverage report
npm run test:e2e           # E2E tests only
```

**Code Quality:**
```bash
npm run lint               # ESLint with auto-fix
npm run format             # Prettier formatting
```

## Architecture Patterns

### Global Services Pattern
Both `PrismaService` and `RedisService` are configured as **global modules** (`@Global()` decorator) and automatically available across all modules without explicit imports. This follows the DRY principle.

**Key locations:**
- `src/prisma/prisma.module.ts` - Global Prisma module
- `src/redis/redis.module.ts` - Global Redis module

**Usage in any service:**
```typescript
constructor(
  private prisma: PrismaService,
  private redis: RedisService,
) {}
```

### Configuration Management

Environment variables follow a **layered loading strategy** with multi-environment support:

**File Structure:**
- `.env.example` - Template with all variables and documentation (committed)
- `.env.development` - Development defaults without secrets (committed)
- `.env.development.local` - Local overrides with real credentials (gitignored)
- `.env.production` - Production template with placeholders (committed)
- `.env.production.local` - Real production values (gitignored, use deployment platform)
- `.env.test` - Unit test configuration (committed)
- `.env.e2e` - E2E test configuration (committed)

**Loading Priority** (from high to low):
1. System environment variables (deployment platform injection)
2. `.env.{NODE_ENV}.local` (local overrides, not committed)
3. `.env.{NODE_ENV}` (environment defaults, committed)

**Important Notes:**
- `.env` file is **NOT loaded** and should **NOT exist** in the project
- If `.env` file exists, the application will show a warning on startup
- All environment configuration must use `.env.{NODE_ENV}` or `.env.{NODE_ENV}.local` files

**Configuration Object** (`src/config/configuration.ts`):
```typescript
{
  port: number,
  nodeEnv: string,
  database: { url: string },
  redis: { host, port, password, db },
  logging: { level: string },
  api: { prefix, corsOrigin },
  jwt: { secret, expiresIn }
}
```

**Access via ConfigService:**
```typescript
constructor(private configService: ConfigService) {}

const port = this.configService.get<number>('app.port');

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shitgood-bradford54) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
