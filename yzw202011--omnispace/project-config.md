---
trigger: always_on
description: > **此文件为AI编程助手的强制上下文。每次开发会话开始时必须完整读取。**
---

# OmniSpace AI v2.3.1 — AI开发记忆文件

> **此文件为AI编程助手的强制上下文。每次开发会话开始时必须完整读取。**
> **任何与此文件冲突的技术决策均无效。此文件 = 开发铁律。**
> **最后更新：2026-08-12 | 前端框架：React 19.2（无约束选型最终裁定）**

---

## 0. 会话启动检查清单

每次开始编码前，AI必须确认以下全部为YES：

- [ ] 我已读取此文件全部内容
- [ ] 我知道前端用React 19.2，不是Vue3，不是Svelte，不是SolidJS
- [ ] 我知道状态管理用Zustand，不是Pinia，不是Redux
- [ ] 我知道包管理用pnpm，不是npm，不是yarn
- [ ] 我知道3D渲染用React Three Fiber，不是原生Three.js
- [ ] 我知道Python包管理用uv，不是pip
- [ ] 我知道后端用FastAPI + Pydantic v2，不是Django/Flask
- [ ] 我知道桌面壳层用Tauri 2.x (Rust)，不是Electron
- [ ] 我知道数据库用SQLite + ChromaDB，不是PostgreSQL/MongoDB
- [ ] 我知道所有CSS颜色必须用var(--color-*)，禁止硬编码
- [ ] 我知道在进行所有项目操作和决策过程中，必须严格遵循团队协作模式。团队成员应根据项目需求明确各自承担的职业角色，包括但不限于产品经理、设计师、前端开发工程师、后端开发工程师、测试工程师等。每个角色需从其专业视角出发思考问题，确保在需求分析、方案设计、开发实现、测试验证等各个环节都能充分发挥专业优势。团队成员之间需保持持续有效的沟通，定期召开协作会议，共享信息，解决分歧，确保项目目标一致且各项工作有序推进。所有决策应基于团队共同讨论，充分考虑各角色的专业意见，形成科学合理的实施方案。
**如有任何一项为NO，立即停止编码并重新读取此文件。**

---

## 1. 技术栈锁定表（LOCKED — 禁止变更）

| 层级 | 锁定技术 | 禁止使用 | 锁定方式 |
|------|---------|---------|---------|
| 前端框架 | **React 19.2** (Function Component + Hooks + Compiler 1.0) | Vue3, Svelte, SolidJS, Angular | package.json |
| 编译优化 | **React Compiler 1.0** | 手动useMemo/useCallback（99%场景不需要） | babel.config.js |
| 前端包管理 | **pnpm 9+** | npm, yarn | pnpm-lock.yaml |
| 状态管理 | **Zustand 5** (slice模式) | Pinia, Redux, MobX, Recoil | package.json |
| 前端路由 | **React Router 7** | Vue Router, TanStack Router | package.json |
| 虚拟列表 | **@tanstack/react-virtual** | vue-virtual, react-window | package.json |
| 3D渲染 | **React Three Fiber + @react-three/drei** | 原生Three.js, TresJS, Babylon.js | package.json |
| 组件库基座 | **shadcn/ui (Base UI)** | Element Plus, Ant Design, MUI | components/ui/ |
| 拖拽交互 | **dnd-kit** | react-dnd, vue-draggable | package.json |
| AI流式 | **Vercel AI SDK** (useChat/useCompletion) | 手写SSE解析 | package.json |
| 错误边界 | **React Error Boundaries + Suspense** | Vue errorHandler | - |
| 类型系统 | **TypeScript 5.x** (strict: true) | JavaScript, CoffeeScript | tsconfig.json |
| 运行时校验 | **Zod 4.x** | io-ts, yup, joi | package.json |
| CSS框架 | **Tailwind CSS 4** | Styled Components, CSS-in-JS | - |
| 图标库 | **lucide-react** | lucide-vue-next, heroicons, font-awesome | - |
| 图表库 | **ECharts** (echarts-for-react) | Chart.js, D3(仅知识图谱) | - |
| 构建工具 | **Vite 6+** | Webpack, Rollup, esbuild直接使用 | vite.config.ts |
| 后端语言 | **Python 3.12** | Node.js, Go, Java | - |
| 后端框架 | **FastAPI** | Django, Flask, Tornado | - |
| Python包管理 | **uv** | pip, poetry, conda | requirements.txt |
| AI推理 | **PyTorch 2.8+** | TensorFlow, JAX | - |
| 数据校验 | **Pydantic v2** | dataclasses(仅简单场景), marshmallow | - |
| 关系数据库 | **SQLite** (WAL模式 + FTS5) | PostgreSQL, MySQL | - |
| 向量数据库 | **ChromaDB** | Pinecone, Weaviate, Milvus | - |
| 任务队列 | **Celery + Redis** | RQ, Huey, Dramatiq | - |
| 桌面壳层 | **Tauri 2.x** (Rust) | Electron, NW.js | - |
| C/C++构建 | **CMake 3.25+ + vcpkg** | Make, Bazel | - |
| 浏览器内核 | **CEF 120+** | WebView2(仅Windows回退) | - |
| 视频编码 | **FFmpeg 7.x** | libav直接使用 | - |

---

## 2. 前端编码铁律（React 19.2）

### 2.1 组件规范

```
组件文件扩展名：.tsx（禁止.vue, .jsx）
组件命名：PascalCase（OmniButton.tsx）
页面组件：XxxPage.tsx
全局组件前缀：Omni（OmniButton, OmniModal, OmniDrawer）
组件结构：Function Component + Hooks（禁止Class Component）
```

### 2.2 组件模板

```tsx
// 标准组件写法 — 严格遵守
import { useState, useEffect, useMemo, useCallback } from 'react';

interface OmniButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

export function OmniButton({ variant = 'primary', size = 'md', children, onClick }: OmniButtonProps) {
  // 1. State
  // 2. Effects (MUST have cleanup return)
  useEffect(() => {
    const ws = new WebSocket('ws://localhost:8000/ws');
    return () => ws.close(); // MUST cleanup
  }, []);
  // 3. Memoized values (仅性能瓶颈处使用，Compiler自动处理大部分)
  // 4. Event handlers
  // 5. Render
  return <button className={`btn btn-${variant} btn-${size}`} onClick={onClick}>{children}</button>;
}
```

### 2.3 前端绝对禁止

| 禁止 | 正确替代 |
|------|---------|
| `any` 类型 | `unknown` + 类型守卫 / Zod schema |
| 手动useMemo/useCallback（99%场景） | React Compiler自动处理 |
| 硬编码颜色 `#3b82f6` | `var(--color-primary)` |
| `!important` | 提升选择器特异性 |
| 全量引入UI库 | Tree-shaking按需引入 |
| useEffect无cleanup | MUST return cleanup函数 |
| 嵌套回调>2层 | async/await |
| class组件 | Function Component + Hooks |
| Redux/Pinia/MobX | Zustand 5 |
| 直接操作数据库 | 调后端API |
| WebSocket传AI token | SSE（Server-Sent Events） |
| 动画 width/height/top/left/margin | transform/opacity |
| 超过100条列表不虚拟化 | @tanstack/react-virtual |

### 2.4 状态管理（Zustand）

```tsx
// 10个store: useAppStore, useChatStore, usePaintStore, useStoryboardStore,
// useLearningStore, useModelStore, useStyleStore, useSettingsStore,
// useModalStore, useWebSocketStore
import { create } from 'zustand';

interface AppState {
  activeModule: string;
  setActiveModule: (module: string) => void;
}

export const useAppStore = create<AppState>((set) => ({
  activeModule: 'chat',
  setActiveModule: (module) => set({ activeModule: module }),
}));
```

### 2.5 路由（React Router 7）

8个页面路由：
```
/chat     → ChatPage
/paint    → PaintPage
/comic    → ComicPage（含3D导演台子路由）
/learn    → LearnPage
/models   → ModelsPage
/style    → StylePage
/settings → SettingsPage
/about    → AboutPage
```

### 2.6 3D导演台（React Three Fiber）

```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yzw202011/OmniSpace](https://github.com/Yzw202011/OmniSpace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
