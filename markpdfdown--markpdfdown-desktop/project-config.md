---
trigger: always_on
description: npm run dev                    # Run full dev environment (generates Prisma, starts Electron with IPC)
---

# AGENTS.md - MarkPDFdown Desktop Development Guide

## Build Commands

```bash
# Development
npm run dev                    # Run full dev environment (generates Prisma, starts Electron with IPC)

# Production Build
npm run build                  # Full production build (generates Prisma + builds Electron app)
npm run start                  # Preview production build (generates Prisma + previews Electron)

# Platform-specific builds (must run npm run build first)
npm run build:win              # Build Windows NSIS installer (x64 + arm64)
npm run build:mac              # Build macOS DMG (x64 + arm64)
npm run build:linux            # Build Linux AppImage (x64 + arm64)

# Database
npm run generate               # Generate Prisma client (required after schema changes)
npm run migrate:dev            # Run migrations in development (creates/updates database)
npm run migrate:reset          # Reset database and re-run all migrations (WARNING: deletes all data)

# Other
npm run lint                   # Run ESLint with auto-fix on all .js/.jsx/.ts/.tsx files
npm run logo                   # Generate app icons from src/renderer/assets/logo.png to public/icons/
```

## Environment Setup

### Prerequisites
- **Node.js**: v18+ recommended (ESM support required)
- **npm**: v8+ (comes with Node.js)
- **Git**: For version control
- **Platform-specific tools**:
  - Windows: No additional tools needed
  - macOS: Xcode Command Line Tools
  - Linux: Standard build tools (`build-essential` on Debian/Ubuntu)

### First-time Setup
```bash
# Clone repository
git clone <repository-url>
cd markpdfdown-desktop

# Install dependencies
npm install

# Generate Prisma client
npm run generate

# Run database migrations
npm run migrate:dev

# Start development server
npm run dev
```

## Testing

本项目已配置完整的测试套件，使用 Vitest 作为测试框架。

### 测试命令
```bash
# 运行所有测试
npm test

# 运行单元测试（main/server）
npm run test:unit

# 运行渲染进程测试（React 组件）
npm run test:renderer

# 监听模式（开发时使用）
npm run test:watch

# 生成覆盖率报告
npm run test:coverage
```

### 详细文档
完整的测试指南请参阅: **[docs/TESTING_GUIDE.md](./docs/TESTING_GUIDE.md)**

包含内容:
- 测试框架和工具介绍
- 所有测试文件详细说明
- 测试模式和最佳实践
- Mock 策略和测试隔离
- 故障排查指南
- 覆盖率目标和成功标准

## Code Style Guidelines

### TypeScript
- **Strict Mode**: Enabled globally - no `any` unless explicitly allowed by ESLint config
- **noUnusedLocals/noUnusedParameters**: 
  - Frontend (`tsconfig.app.json`): Enabled - remove unused declarations
  - Backend (`tsconfig.backend.json`): Not explicitly enabled
- **Module System**: ESM (type: "module" in package.json)
- **Module Alias**: 
  - `@` alias available in frontend only (via bundler resolution)
  - Backend uses relative imports (`../`, `./`) and `.js` extensions in imports

### ESLint Configuration
- **Config File**: `eslint.config.js` (flat config format, not `.eslintrc.cjs`)
- Extends TypeScript-ESLint recommended config
- Allows `any` type (`@typescript-eslint/no-explicit-any`: "off")
- Allows async promise executors (`no-async-promise-executor`: "off")
- React hooks enforcement enabled
- React refresh warnings enabled (allows constant exports)
- Ignores: `dist/**/*`, `release/**/*`

### Naming Conventions
- **Files**: 
  - Controllers: PascalCase (e.g., `TaskController.ts`, `ProviderController.ts`, `FileController.ts`)
  - DAL: PascalCase (e.g., `TaskDal.ts`, `ProviderDal.ts`, `modelDal.ts` - note: some use camelCase)
  - Logic: PascalCase (e.g., `Task.ts`, `File.ts`) or camelCase (e.g., `model.ts`)
  - Routes: PascalCase (e.g., `Routes.ts`)
  - **Recommendation**: Use PascalCase for consistency
- **Variables/Functions**: camelCase (e.g., `createTasks`, `getAllTasks`)
- **Constants**: SCREAMING_SNAKE_CASE for config values, camelCase for local constants
- **Classes/Interfaces**: PascalCase (e.g., `Task`, `Provider`)
- **Database Models**: PascalCase singular (e.g., `Provider`, `Task`, `Model`, `TaskDetail`)
- **DAL exports**: Default export object with methods (e.g., `export default { findAll, create, ... }`)


### Backend Architecture (Clean Architecture)
The backend follows clean architecture principles with clear separation of concerns, organized into four layers:

#### Layer Structure
```
src/core/
├── infrastructure/     # External dependencies (database, config, adapters)
│   ├── db/            # Prisma database client and migrations
│   ├── config/        # Worker configuration
│   ├── services/      # Infrastructure services (FileService)
│   └── adapters/      # External service adapters
│       ├── llm/       # LLM client implementations (OpenAI, Anthropic, etc.)
│       └── split/     # File splitter implementations (PDFSplitter, ImageSplitter)
├── application/       # Application-specific business logic
│   ├── services/      # Application services (WorkerOrchestrator, ModelService)
│   └── workers/       # Background processing workers
├── domain/            # Core business logic (interfaces and pure logic only)
│   ├── repositories/  # Data access layer
│   ├── split/         # Splitter interface and pure logic (ISplitter, PageRangeParser)
│   └── llm/           # LLM client interface and types (ILLMClient)
└── shared/            # Cross-cutting concerns
    └── events/        # Event bus for worker coordination
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarkPDFdown/markpdfdown-desktop](https://github.com/MarkPDFdown/markpdfdown-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
