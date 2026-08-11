---
trigger: always_on
description: > 项目完整上下文（技术栈、目录结构、架构、数据流等）见 [CONTEXT.md](CONTEXT.md)
---

# 项目代码规范

> 项目完整上下文（技术栈、目录结构、架构、数据流等）见 [CONTEXT.md](CONTEXT.md)

## TypeScript 类型规范

### 禁止使用 `any` 类型
- ❌ **禁止**: 使用 `any` 类型
- ✅ **推荐**: 使用具体类型或 `unknown` 类型

**错误示例**:
```typescript
const data: any = fetchData()
const window = (window as any).__TAURI__
```

**正确示例**:
```typescript
interface WindowWithTauri extends Window {
  __TAURI__?: unknown
}
const window = (window as WindowWithTauri).__TAURI__

// 或使用类型断言
const data = fetchData() as DataType
```

### 组件 Ref 类型定义
使用 `InstanceType<typeof ComponentName>` 获取组件实例类型，而非手动定义接口：

**错误示例**:
```typescript
const conversationRef = ref<{ scrollToBottom: (options?: { behavior?: 'smooth' | 'auto' }) => void } | null>(null);
```

**正确示例**:
```typescript
const conversationRef = ref<InstanceType<typeof ConversationView>>();
```

### 类型定义要求
- 所有函数参数必须有明确的类型注解
- 所有函数返回值必须有明确的类型注解
- 接口和类型定义必须使用 `interface` 或 `type` 关键字

## 代码清理规范

### 保留未使用的导入
- ✅ **保留**: 可能会在运行时动态导入的模块
- ✅ **保留**: 类型定义的导入

**示例**:
```typescript
// ✅ 保留 - 动态导入
const { open } = await import('@tauri-apps/plugin-dialog')

// ✅ 保留 - 类型定义
import type { DefineComponent } from 'vue'
```

### 代码清理原则
- 只删除确认完全不会被使用的代码
- 对于条件性使用的代码，需要检查所有执行路径
- 动态导入的模块不要删除导入语句

## 文件组织规范

### 文档路径规范
- 在 `AGENTS.md`、`CONTEXT.md`、`docs/`、`changelog/` 等仓库文档中引用项目文件时，统一使用**仓库相对路径**
- ❌ 禁止写项目绝对路径，禁止在文档中出现任何本机绝对路径示例
- ✅ 推荐写法：`src/components/BChatSidebar/components/InputToolbar.vue`
- 如果需要 Markdown 链接，链接目标也使用相对路径

### 代码与测试路径规范
- 所有源码、测试代码、注释中**禁止硬编码本机绝对路径**（如 `/Users/<用户名>/...`、`C:\Users\<用户名>\...`）
- ❌ 反例：`find /Users/<用户名> -name widget.json`、`cat /Users/<用户名>/.ssh/config`
- ✅ 正例：使用 POSIX 通用路径 `/home/user/...`、`os.homedir()`、`os.tmpdir()`、`process.cwd()`、临时 fixture 目录或仓库相对路径
- 测试用例需要「工作区外路径」做越权读取测试时，统一使用 `/home/user/...`（macOS/Linux 通用）而非本机真实家目录
- 需要可执行的真实文件系统操作时，使用 `os.tmpdir()` 拼接临时目录，并在测试结束后清理

### 组件引入规范
- **B 开头的组件**已通过 `unplugin-vue-components` 全局自动引入，无需手动 import
- 手动引入场景：类型定义、动态导入、编辑器内置组件

### 示例
```typescript
// ✅ 无需手动引入（已全局注册）
import { BButton, BModal } from '@/components'

// ✅ 需要手动引入的场景
import type { BButtonProps } from '@/components' // 类型定义
const { open } = await import('@/components/BModal') // 动态导入
```

## 注释规范

### 基本要求
- **所有代码必须有注释**，不允许出现无注释的函数、类、接口或复杂逻辑块
- 注释必须准确描述代码的意图，而不是简单重复代码本身
- 注释需要随代码同步更新，禁止出现过时或误导性注释

### 文件头注释
每个文件顶部必须包含文件说明注释：

```typescript
/**
 * @file 文件名
 * @description 文件功能描述
 */
```

### 函数 / 方法注释
所有函数和方法必须使用 JSDoc 格式注释：

```typescript
/**
 * 获取用户信息
 * @param userId - 用户 ID
 * @returns 用户信息对象，不存在时返回 null
 */
async function getUserInfo(userId: string): Promise<UserInfo | null> {
  // ...
}
```

### 接口 / 类型注释
所有接口和类型定义必须添加说明：

```typescript
/**
 * 用户信息
 */
interface UserInfo {
  /** 用户唯一标识 */
  id: string
  /** 用户名 */
  name: string
  /** 注册时间（时间戳） */
  createdAt: number
}
```

### 复杂逻辑注释
对于复杂的业务逻辑，必须在关键步骤添加行内注释：

```typescript
async function syncData() {
  // 1. 从本地缓存读取上次同步时间
  const lastSyncTime = await getLastSyncTime()

  // 2. 拉取服务端增量数据
  const delta = await fetchDelta(lastSyncTime)

  // 3. 合并数据，以服务端为准（本地修改会被覆盖）
  const merged = mergeData(localData, delta)

  // 4. 写入本地数据库并更新同步时间
  await saveLocal(merged)
  await updateLastSyncTime(Date.now())
}
```

### 临时代码 / 待办注释
- 使用 `// TODO:` 标记待完成的功能
- 使用 `// FIXME:` 标记已知问题需要修复
- 使用 `// HACK:` 标记临时方案，需说明原因和后续处理计划

```typescript
// TODO: 后续支持批量操作
// FIXME: 当列表为空时会触发越界错误
// HACK: 临时绕过接口限制，等后端修复后移除
```

### 禁止事项
- ❌ 禁止无意义注释，如 `// 定义变量 i`
- ❌ 禁止注释掉的废弃代码长期存在，应直接删除
- ❌ 禁止用注释代替清晰的命名，命名本身应具有可读性

## 代码检查规范

### ESLint 检查
- **所有代码必须通过 ESLint 检查**，提交前执行 `pnpm lint` 修复问题
- 配置文件：`.eslintrc.cjs`，忽略文件：`.eslintignore`
- 检查范围：`src/` 下 `.vue`、`.ts`、`.tsx`、`.js`、`.jsx` 文件
- 核心规则（来自已有配置）：
  - `@typescript-eslint/no-explicit-any`：禁止使用 `any`（见[TypeScript 类型规范](#typescript-类型规范)）
  - `import/order`：强制 import 排序（vue/vue-router/pinia 优先，`@/` 次之，按字母升序）
  - `vue/order-in-components`：强制组件选项顺序
  - `vue/attributes-order`：强制组件属性排序
  - `vue/html-self-closing`：强制自闭合标签风格
  - `vue/component-name-in-template-casing`：模板中组件名使用 PascalCase
  - `no-console`：关闭（Electron 桌面应用允许 console）

### Stylelint 检查
- **所有样式代码必须通过 Stylelint 检查**，提交前执行 `pnpm lint:style` 修复问题
- 配置文件：`.stylelintrc.cjs`
- 检查范围：`src/` 下 `.vue`、`.less`、`.css` 文件
- 核心规则（来自已有配置）：
  - CSS 属性书写顺序遵循 `stylelint-config-recess-order`
  - Less 文件使用 `postcss-less` 语法解析
  - Vue 文件使用 `postcss-html` 语法解析
  - 允许 `v-deep`、`:global`、`fade()` 等 Vue/Less 特殊语法
  - 不限制选择器类名格式（UnoCSS 原子化类名不受约束）

### TypeScript 类型检查
- **所有代码必须通过 TypeScript 类型检查**，提交前执行 `pnpm exec tsc --noEmit`
- 使用 `strict` 模式（`tsconfig.json` 中 `strict: true`）
- 开启 `noUnusedLocals` 和 `noUnusedParameters`
- 项目使用 `@typescript-eslint/*` 替代已废弃的 TSLint，**禁止引入 TSLint**

### 检查命令速查

| 命令 | 说明 |
|------|------|
| `pnpm lint` | ESLint 检查 + 自动修复 |
| `pnpm lint:style` | Stylelint 检查 + 自动修复 |
| `pnpm exec tsc --noEmit` | TypeScript 类型检查（无输出） |
| `pnpm exec eslint src --ext .vue,.ts,.tsx,.js,.jsx` | ESLint 仅检查（不修复） |
| `pnpm exec stylelint 'src/**/*.{vue,less,css}'` | Stylelint 仅检查（不修复） |

## 工具库使用规范

### lodash-es 优先
项目已安装 `lodash-es` 和 `@types/lodash-es`，**优先使用 lodash-es 替代手写工具函数**，**禁止使用 `lodash`**（CommonJS 版本不利于 tree-shaking）。

**应使用 lodash-es 的场景**：

| 手写模式 | lodash-es 替代 | 说明 |
|----------|---------------|------|
| 手写 `debounce` / `throttle` | `import { debounce, throttle } from 'lodash-es'` | 防抖 / 节流 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xbinator/tibis](https://github.com/xbinator/tibis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
