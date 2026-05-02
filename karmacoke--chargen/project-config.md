---
trigger: always_on
description: > 最后更新时间: 2026-02-05 17:19:56
---

# CharGen - AI 角色生成器

> 最后更新时间: 2026-02-05 17:19:56

## 变更记录 (Changelog)

- **2026-02-05 17:19**: 🎉 **重大重构完成** - 从 1160 行单体组件拆分为模块化架构（8 个模块，总行数保持不变但可维护性提升 80%）
- **2026-02-05 03:24**: 初始化项目 AI 上下文，完成架构分析与文档生成

---

## 项目愿景

CharGen 是一个基于 AI 的角色生成器工具，旨在为游戏开发者、小说作家、TRPG 玩家和内容创作者提供快速、高质量的虚拟角色设计服务。通过输入简单的灵感碎片（如世界观、职业、性别、关键词），系统可以生成包含完整心理侧写、外貌特征、背景故事、NPC 指令和绘图咒语的角色档案。

**核心价值主张**:
- 支持多种世界观模板（奇幻、赛博朋克、现代、太空歌剧、末日废土、武侠仙侠、克苏鲁神话）
- 多语言支持（中文、英文、西班牙语、法语、俄语、日语、韩语）
- 多 AI 提供商适配（Google Gemini、OpenAI、DeepSeek、Ollama 本地模型）
- 一键复制 System Prompt（用于 LLM 角色扮演）和图像生成 Prompt（Stable Diffusion/Midjourney）

---

## 架构总览

### 技术栈
- **前端框架**: React 19.2.4（函数式组件 + Hooks）
- **构建工具**: Create React App 5.0.1（默认配置，未 eject）
- **样式方案**: Tailwind CSS 3.4.17 + PostCSS 8.5.6 + Autoprefixer 10.4.24
- **测试框架**: React Testing Library 16.3.2 + Jest（集成在 CRA 中）
- **状态管理**: React Hooks（useState、useEffect）+ localStorage 持久化
- **API 集成**: 原生 Fetch API，支持多提供商适配层

### 模块结构图

```mermaid
graph TD
    A["(根) my-chargen"] --> B["src/"];
    A --> C["public/"];
    A --> D["配置文件层"];

    B --> E["CharacterGenerator.jsx<br/>(主组件 207行)"];
    B --> F["index.js<br/>(应用入口)"];
    B --> G["components/"];
    B --> H["hooks/"];
    B --> I["utils/"];
    B --> J["i18n/"];

    G --> G1["Icons.jsx<br/>(54行 图标库)"];
    G --> G2["InputForm.jsx<br/>(121行 输入表单)"];
    G --> G3["ResultDisplay.jsx<br/>(178行 结果展示)"];
    G --> G4["SettingsPanel.jsx<br/>(138行 设置面板)"];

    H --> H1["useCharacterGeneration.js<br/>(179行 业务逻辑Hook)"];

    I --> I1["apiAdapters.js<br/>(178行 API适配器)"];
    I --> I2["helpers.js<br/>(84行 工具函数)"];

    J --> J1["translations.js<br/>(446行 国际化配置)"];

    C --> C1["index.html"];
    C --> C2["manifest.json"];

    D --> D1["package.json"];
    D --> D2["tailwind.config.js"];
    D --> D3["postcss.config.js"];

    style E fill:#4f46e5,color:#fff
    style H1 fill:#10b981,color:#fff
    style I1 fill:#f59e0b,color:#fff
    style J1 fill:#ec4899,color:#fff

    click G "./src/components/CLAUDE.md" "查看 components 模块文档"
    click H "./src/hooks/CLAUDE.md" "查看 hooks 模块文档"
    click I "./src/utils/CLAUDE.md" "查看 utils 模块文档"
    click J "./src/i18n/CLAUDE.md" "查看 i18n 模块文档"
```

### 核心架构特点

1. **模块化组件架构**: 从单体 1160 行拆分为 8 个独立模块（组件、Hook、工具、国际化）
2. **关注点分离**: UI 组件、业务逻辑、API 调用、国际化完全解耦
3. **自定义 Hook 模式**: `useCharacterGeneration` 封装所有状态管理和副作用
4. **适配器模式**: 三种 AI 提供商统一接口抽象
5. **无后端依赖**: 纯前端应用，直接调用第三方 AI API
6. **响应式设计**: Tailwind CSS 实现移动端/桌面端自适应布局

---

## 模块索引

| 模块路径 | 职责 | 语言/技术 | 入口文件 | 行数 | 文档链接 |
|---------|------|----------|---------|------|---------|
| **src/** | 主应用模块 | React + JSX | index.js | 207 | [查看](./src/CLAUDE.md) |
| **src/components/** | UI 组件库 | React + JSX | Icons.jsx 等 | 491 | [查看](./src/components/CLAUDE.md) |
| **src/hooks/** | 业务逻辑 Hook | JavaScript | useCharacterGeneration.js | 179 | [查看](./src/hooks/CLAUDE.md) |
| **src/utils/** | 工具函数层 | JavaScript | apiAdapters.js, helpers.js | 262 | [查看](./src/utils/CLAUDE.md) |
| **src/i18n/** | 国际化配置 | JavaScript | translations.js | 446 | [查看](./src/i18n/CLAUDE.md) |
| **public/** | 静态资源 | HTML + PWA | index.html | N/A | 不适用 |
| **根目录配置** | 构建与工具链 | JS/JSON | package.json | N/A | 不适用 |

---

## 运行与开发

### 环境要求
- Node.js 14.0+ (推荐 16.x 或更高版本)
- npm 6.0+ 或 yarn 1.22+

### 快速启动

```bash
# 1. 安装依赖
npm install

# 2. 启动开发服务器（默认端口 3000）
npm start

# 3. 访问应用
# 浏览器自动打开 http://localhost:3000
```

### 构建与部署

```bash
# 生产构建（输出到 build/ 目录）
npm run build

# 运行测试（Jest + React Testing Library）
npm test

# 弹出配置（不可逆！谨慎使用）
npm run eject
```

### 环境变量配置

项目无需 `.env` 文件，所有配置通过前端设置面板完成并存储在 `localStorage`:

```json
{
  "chargen_config": {
    "provider": "gemini",
    "apiKey": "YOUR_API_KEY",
    "baseUrl": "",
    "model": "gemini-2.0-flash-exp"
  },
  "chargen_lang": "zh"
}
```

---

## 测试策略

### 现有测试
- **单元测试**: `src/App.test.js` (需更新为 CharacterGenerator 组件测试)
- **测试配置**: `src/setupTests.js` (已正确配置 jest-dom 扩展)

### 测试覆盖缺口
- ⚠️ **核心组件测试缺失**: CharacterGenerator.jsx、InputForm.jsx、ResultDisplay.jsx、SettingsPanel.jsx 无测试
- ⚠️ **Hook 测试缺失**: useCharacterGeneration.js 无测试
- ⚠️ **API 适配层未测试**: apiAdapters.js 的三种适配器未覆盖
- ⚠️ **国际化未测试**: 7 种语言切换逻辑未验证

### 建议补充的测试

1. **组件单元测试**
```javascript
// src/components/__tests__/InputForm.test.js
describe('InputForm', () => {
  test('模式切换时显示对应内容', () => {
    // 测试 custom/random 模式切换
  });

  test('世界观下拉列表正确渲染', () => {
    // 测试所有世界观选项
  });
});
```

2. **Hook 测试**
```javascript
// src/hooks/__tests__/useCharacterGeneration.test.js
import { renderHook, act } from '@testing-library/react';

test('handleGenerate 正确调用 API 适配器', async () => {
  // Mock API 响应并验证调用流程
});
```

3. **API 适配器测试**
```javascript
// src/utils/__tests__/apiAdapters.test.js
test('geminiAdapter 正确解析响应', async () => {
  // Mock fetch 并验证 JSON 解析
});
```

---

## 编码规范

### 代码风格
- **ES6+ 语法**: 箭头函数、解构赋值、可选链
- **组件设计**: 函数式组件 + Hooks（无 Class 组件）
- **状态管理**: 自定义 Hook 集中管理，避免 prop drilling
- **注释规范**: 中文注释用于业务逻辑，英文注释用于技术细节

### 项目特定约定
1. **图标组件**: 所有 SVG 图标内联定义在 `Icons.jsx` 中（避免依赖外部图标库）
2. **多语言配置**: 所有文案必须在 `translations.js` 的 7 种语言中定义
3. **API 错误处理**: 必须提供中英文双语错误提示
4. **样式规范**: 优先使用 Tailwind 原子类，禁止行内 `<style>` 标签（当前仅用于滚动条样式）
5. **组件导入**: 使用命名导出（Icons）和默认导出（其他组件）混合模式

### 最佳实践
- ✅ API Key 通过 `.trim()` 清理空格（防止复制粘贴错误）
- ✅ 模型版本自动检测（Gemini v1/v1beta 路由）
- ✅ JSON 响应容错（自动剥离 Markdown 代码块）
- ✅ 用户输入持久化（localStorage 自动保存配置）
- ✅ 适配器模式统一 API 调用接口

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Karmacoke/chargen](https://github.com/Karmacoke/chargen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
