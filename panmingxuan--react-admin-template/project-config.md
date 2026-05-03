---
trigger: always_on
description: > 本文档整合代码约束设计思想与行为规则，
---

# 🧠 前端项目 AI 协作开发规范

> 本文档整合代码约束设计思想与行为规则，  
> 结合现代前端开发最佳实践，用于指导 AI 在前端项目中的代码生成与协作行为。

---

## 📐 第一部分：架构设计原则（Architecture Principles）

### 一、组件单一职责原则

> 每个组件只做一件事，做好一件事。

**实践**

- 展示组件与容器组件分离
- 布局组件与业务组件解耦
- 一个文件一个导出组件（除内部辅助组件）

### 二、类型契约优先原则

> Props 即契约，类型即文档。

**实践**

- 所有组件 Props 必须定义独立类型/接口
- 使用 `export type` 导出类型定义
- 禁止使用 `any`、`unknown` 作为 Props 类型

### 三、依赖隔离原则

> 组件不依赖具体实现，通过 Props 注入依赖。

**实践**

- 路由逻辑通过 Props 回调或 Render Props 注入
- 路由 API（如 TanStack Router）仅在适配器中使用
- 通用组件保持框架无关

### 四、样式一致性原则

> 设计系统即代码规范。

**实践**

- 使用 Tailwind CSS + CSS 变量管理主题
- 遵循 Catalyst Design System 色彩与间距规范
- 暗色模式使用透明度映射而非硬编码颜色

### 五、可访问性优先原则

> 每个交互元素都必须可访问。

**实践**

- 所有 `<button>` 声明 `type` 属性
- 所有 `<img>` 包含有意义的 `alt` 属性
- 所有 `<svg>` 包含 `<title>` 元素
- 使用语义化 HTML 标签

---

## 🧩 第二部分：代码行为规范（Execution Rules）

### 1. TypeScript 规范

```typescript
// ✅ 正确：使用 import type
import type { ReactNode } from 'react';

// ❌ 错误：直接导入类型
import { ReactNode } from 'react';

// ✅ 正确：明确的 Props 类型
interface ButtonProps {
  children: ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

// ❌ 错误：使用 any
interface ButtonProps {
  onClick: any;
}
```

### 2. React/JSX 规范

```tsx
// ✅ 正确：Hook 在组件顶层
function Component() {
  const [state, setState] = useState(false);
  // ...
}

// ❌ 错误：条件内使用 Hook
function Component({ show }) {
  if (show) {
    const [state, setState] = useState(false); // 错误！
  }
}

// ✅ 正确：使用唯一标识作为 key
{items.map((item) => (
  <Item key={item.id} {...item} />
))}

// ❌ 错误：使用数组索引作为 key
{items.map((item, index) => (
  <Item key={index} {...item} />
))}
```

### 3. 样式规范

```tsx
// ✅ 正确：使用语义化的 Tailwind 类名组合
<button className="rounded-lg bg-zinc-950/5 px-3 py-2 text-sm font-medium text-zinc-900 hover:bg-zinc-950/10">

// ❌ 错误：内联样式
<button style={{ backgroundColor: 'rgba(0,0,0,0.05)' }}>

// ✅ 正确：暗色模式使用透明度
<div className="bg-white dark:bg-zinc-900 border-zinc-950/5 dark:border-white/10">

// ❌ 错误：硬编码暗色值
<div className="bg-white dark:bg-[#18181b]">
```

### 4. 组件结构规范

```tsx
// ✅ 正确：Props 接口 + 函数声明
interface SidebarProps {
  items: NavItem[];
  currentPath?: string;
  onNavigate?: (path: string) => void;
}

export function Sidebar({ items, currentPath, onNavigate }: SidebarProps) {
  // 实现
}

// ❌ 错误：匿名导出 + 内联类型
export default ({ items }: { items: any[] }) => {
  // ...
}
```

### 5. 文件组织规范

```
/components
  /layout
    sidebar.tsx          # 主组件
    styles.ts            # 样式常量
    types.ts             # 类型定义
    index.ts             # 统一导出
    /adapters            # 路由适配器
  /ui                    # 基础 UI 组件（shadcn/ui + Catalyst wrapper）
    /shadcn              # shadcn/ui 原语层（CLI 生成，不直接修改）
      button.tsx
      input.tsx
      dialog.tsx
      ...
    /button              # Button wrapper 目录
      button.tsx         # Catalyst 风格封装
      types.ts           # Props 类型定义
      index.ts           # 统一导出
    /input               # Input wrapper 目录（同上结构）
    /dialog              # Dialog wrapper 目录（同上结构）
    button.tsx           # 顶层入口 re-export
    input.tsx            # 顶层入口 re-export
    dialog.tsx           # 顶层入口 re-export
```

---

## 🎨 第三部分：UI 设计准则引用

> 详细的 UI 设计规范请参阅 `doc/UI_DESIGN_SPEC.md`

**核心要点速览：**

- **色系**：Zinc 中性灰 + 透明度（Alpha）
- **圆角**：`rounded-lg` (8px) 作为标准
- **间距**：`px-3 py-2` 紧凑，`px-6 py-8` 宽松
- **图标**：20px（`size-5`），1.5px 线宽
- **分割线**：`border-zinc-950/5` 极淡透明

---

## 🔍 第四部分：UI 问题排查引用

> 详细的 UI 问题排查指南请参阅 `doc/UI_TROUBLESHOOTING.md`

**排查流程速览：**

```
1. 控制台错误 → 2. 元素检查 → 3. 计算样式 → 4. 布局调试 → 5. 代码审查
```

**常见问题分类：**

| 问题类型 | 排查方向 |
|---------|---------|
| 布局错位 | Flexbox/Grid、overflow、position |
| 样式不生效 | 类名拼写、优先级、Tailwind 配置 |
| 暗色模式 | dark: 变体、透明度映射 |
| 交互状态 | hover/focus/active、z-index |
| 响应式 | 断点设置、容器宽度 |

**快速诊断提示词：**

```markdown
请帮我排查以下 UI 问题：
- 问题描述：[现象]
- 期望效果：[目标]
- 相关文件：[路径]
请根据 doc/UI_DESIGN_SPEC.md 和 doc/UI_TROUBLESHOOTING.md 进行分析。
```

---

## ⚙️ 第五部分：AI 协作策略

### 生成前检查

1. 扫描现有组件模式与命名约定
2. 识别项目使用的设计系统
3. 本项目使用 React + Vite + TanStack Router

### 生成中遵守

1. 遵循本文档所有规范
2. 参考 `doc/UI_DESIGN_SPEC.md` 设计准则
3. 遇到 UI 问题参考 `doc/UI_TROUBLESHOOTING.md` 排查指南
4. 保持与现有代码风格一致

### 生成后验证

1. 类型检查通过
2. 无 ESLint/Biome 错误
3. 组件可独立运行测试
4. UI 显示符合设计规范

---

## 🛠️ 第六部分：cn 函数与样式工具

> 基于 `tailwind-merge` + `clsx`，遵循 Tailwind CSS v4 最佳实践

### 核心理念：CSS-First 配置

Tailwind CSS v4 推荐将设计 tokens 定义在 CSS 中而非 JavaScript：

```css
/* src/styles/globals.css */
@import "tailwindcss";

@theme {
  /* 字体系统 */
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-display: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-mono: ui-monospace, "SF Mono", "Menlo", monospace;

  /* 品牌色 - 使用 OKLCH 现代颜色空间 */
  --color-brand: oklch(0.55 0.2 250);
  --color-brand-light: oklch(0.75 0.15 250);
  --color-brand-dark: oklch(0.35 0.2 250);

  /* 语义化颜色 */
  --color-success: oklch(0.65 0.2 145);
  --color-warning: oklch(0.75 0.18 85);
  --color-error: oklch(0.55 0.22 25);
  --color-info: oklch(0.6 0.2 250);
}
```

### Catalyst 设计系统核心参数

> 本项目基于 Catalyst Design System，使用 `Zinc` 色系 + 透明度(Alpha) 设计理念

#### 布局参数

| 组件部分 | Tailwind Class | 设计意图 |
|---------|----------------|----------|
| Sidebar 宽度 | `w-72` (288px) | 比传统 256px 更宽，更现代 |
| 分割线 | `border-zinc-950/5` | 极淡透明黑，非实色灰 |
| 内边距（宽松） | `px-6 py-8` | 留白充足，呼吸感强 |
| 内边距（紧凑） | `px-3 py-2` | 导航项/按钮标准间距 |
| 标准圆角 | `rounded-lg` (8px) | 组件标准圆角 |
| 卡片圆角 | `rounded-2xl` (16px) | 卡片/对话框圆角 |

#### 状态样式映射


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panmingxuan/react-admin-template](https://github.com/panmingxuan/react-admin-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
