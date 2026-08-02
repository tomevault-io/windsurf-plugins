---
trigger: always_on
description: > **注意**：本文档为内部开发规范，面向项目维护者和 AI 助手，最终用户请参考官方文档 https://esmx.dev
---

> **注意**：本文档为内部开发规范，面向项目维护者和 AI 助手，最终用户请参考官方文档 https://esmx.dev

# Claude 开发指令集

> **说明**：本文件为 Claude Code AI 助手的专业开发指令集，用于规范代码生成、错误处理、项目结构等技术标准。

## 🌟 核心原则

**代码简洁性、安全性、自解释性**
- 编写简洁、技术性强的代码，提供准确示例
- 使用面向对象 + 函数式编程模式
- 优先选择迭代和模块化，避免代码重复
- 使用描述性变量名，带有辅助动词（如 isLoading, hasError）

## 🌐 语言使用规范

**统一的语言使用标准，确保团队协作和国际化兼容性**

### 用户沟通语言
使用用户的语言（中文用户用中文）：
- 工作流程界面
- 确认提示
- 错误信息

### 代码语言
一律使用英文：
- git commit 信息
- PR 标题和描述  
- 代码注释
- 变量和函数命名

## 💻 TypeScript 编码标准

### ESM 模块系统要求
- **强制使用 ESM (ECMAScript Modules) 进行开发**
- **使用 `import/export` 语句，禁用 `require/module.exports`**
- **Node.js 内置模块必须使用 `node:` 前缀**
- **配置 `"type": "module"` 在 package.json 中**

### Monorepo 开发规范
- **使用 `pnpm workspace` 管理 monorepo 依赖**
- **包间引用使用 workspace 协议：`"workspace:*"`**
- **核心包名遵循命名空间：`@your-org/package-name`**
- **示例项目使用项目内部导入，避免命名空间冲突**
- **共享配置文件放在根目录**
- **包级别的配置继承根配置**
- **examples 目录包含示例项目，支持嵌套结构**
- **构建产物 (`dist/`) 自动排除在 workspace 外**

#### 目录结构
```
project/
├── packages/           # 核心包目录
│   ├── core/
│   │   ├── package.json
│   │   └── src/
│   ├── utils/
│   │   ├── package.json
│   │   └── src/
│   └── ui/
│       ├── package.json
│       └── src/
├── examples/           # 示例项目目录
│   ├── basic-app/
│   │   ├── package.json
│   │   └── src/
│   └── advanced-demo/
│       ├── package.json
│       └── src/
├── package.json (workspace root)
├── pnpm-workspace.yaml
└── tsconfig.json (base config)
```

#### pnpm-workspace.yaml 配置
```yaml
packages:
  - 'packages/*'      # 核心包
  - 'examples/**'     # 示例项目（支持嵌套）
  - '!**/dist/**'     # 排除构建产物
```

#### 包间依赖
```json
{
  "dependencies": {
    "@your-org/core": "workspace:*",
    "@your-org/utils": "workspace:*"
  }
}
```

#### 导入规范
```typescript
// ✅ 核心包导入
import { createUser } from '@your-org/core';
import { logger } from '@your-org/utils';

// ✅ 示例项目导入（使用相对路径或项目别名）
import { createRouter } from '../router';
import { createVueRouter } from '../router-vue';

// ✅ 内部模块导入
import { validateEmail } from '../validators';
import { formatDate } from './utils';
```

### 类型安全要求
- **所有代码使用 TypeScript，优先使用 interfaces 而非 types**
- **❌ 避免 enums，使用 const 对象代替**
- **启用 TypeScript 严格模式以提高类型安全**
- **❌ 避免 `any` 类型，类型不确定时使用 `unknown`**

#### 代码示例

##### ✅ 推荐写法
```typescript
interface UserData {
  id: string;
  email: string;
  isActive: boolean;
}

const PAYMENT_STATUS = {
  PENDING: 'pending',
  COMPLETED: 'completed',
  FAILED: 'failed'
} as const;

// 使用 unknown 而不是 any
const processInput = (input: unknown): UserData => {
  if (typeof input === 'object' && input !== null) {
    return input as UserData;
  }
  throw new Error('Invalid input');
};
```

##### ❌ 避免写法
```typescript
enum PaymentStatus {
  PENDING = 'pending',
  COMPLETED = 'completed', 
  FAILED = 'failed'
}

// 避免使用 any
const processInput = (input: any): UserData => {
  return input;
};
```

### 导入规范
**严格按照 Biome 默认导入顺序规范：**
1. **Bun 导入**：`bun:*` 模块
2. **Node.js 内置模块**：`node:*` 前缀的模块
3. **外部库**：来自 node_modules 的第三方包
4. **Monorepo 包间导入**：`@your-org/*` 命名空间的包
5. **相对导入（同级）**：`./` 开头的导入
6. **相对导入（父级）**：`../` 开头的导入
7. **内部模块（绝对路径）**：当前包内的绝对导入（仅在没有相对路径可用时使用）
8. **类型导入**：`import type` 语句（每个组内分别排列）

**重要要求：**
- **不同导入组之间必须用空行分隔**
- **同组内导入按字母顺序排列**
- **类型导入放在对应组的最后**
- **✅ Bun 和 Node.js 内置模块必须使用命名空间导入**
- **❌ 严格禁止 Node.js 内置模块的具名导入**

```typescript
// ✅ 符合 Biome 导入顺序规范
import fs from 'node:fs/promises';
import path from 'node:path';
import util from 'node:util';

import express from 'express';
import { z } from 'zod';

import { createUser } from '@your-org/core';
import { logger } from '@your-org/utils';

import { config } from './config';
import { database } from './lib/database';

import { validateRequest } from '../middleware';
import { userSchema } from '../schemas';

import { formatDate } from './utils';
import { constants } from './constants';

import type { Request, Response } from 'express';
import type { UserData } from '@your-org/core';
import type { Config } from './types';

// ❌ 禁止的导入模式
import { readFile } from 'node:fs/promises'; // 禁止：具名导入，应该使用命名空间导入
import { join } from 'node:path';            // 禁止：具名导入，应该使用命名空间导入

// ✅ 正确的 Node.js 内置模块导入方式
import fs from 'node:fs/promises';
import path from 'node:path';

// 使用时通过命名空间访问
const content = await fs.readFile('file.txt', 'utf-8');
const fullPath = path.join('/path', 'to', 'file');
const fs = require('fs');                    // 禁止：CommonJS
```

### 导出规范
- **优先使用命名导出**
- **只在绝对必要时使用默认导出**
- **使用 ESM 导出语法**
- **❌ 严格禁止 CommonJS 导出语法**

```typescript
// ✅ 推荐的 ESM 导出
export const createUser = (data: UserData): User => { /* */ };
export const updateUser = (id: string, data: Partial<UserData>): User => { /* */ };
export { validateEmail, formatDate } from './utils';

// ❌ 严格禁止的 CommonJS 导出
module.exports = { createUser, updateUser };
exports.createUser = (data) => { /* */ };
```

## ⚠️ 错误处理规范

### 严格限制 try-catch 使用
- **避免滥用**：禁止"写一段代码就包一个 try-catch"的习惯
- **强正当性要求**：每次使用 try-catch 都必须有充分的正当理由
- **优先 Result 模式**：绝大多数情况下使用 Result 模式处理错误
- **边界层使用**：try-catch 主要用于应用边界层和工具函数封装
- **禁止静默处理**：严禁在 catch 块中静默忽略错误

### ✅ Result 模式（推荐）
```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// 推荐的错误处理方式
const processUser = async (userData: unknown): Promise<Result<User, ValidationError>> => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esmnext/esmx](https://github.com/esmnext/esmx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
