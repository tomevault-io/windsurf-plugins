---
trigger: always_on
description: > 本文件包含 AI 助手需要了解的项目背景、编码风格和工具配置
---

# StoryForge Agent 指南

> 本文件包含 AI 助手需要了解的项目背景、编码风格和工具配置

## 🧠 永久记忆：自动化测试助手

本项目已配置 **Playwright + Chromium** 无头浏览器自动化测试环境，专为 AI 助手设计。

### 快速启动测试

```bash
# 一键截图所有页面
npm run screenshot

# 截图幕前界面
npm run screenshot:front

# 截图幕后界面
npm run screenshot:back

# 运行完整测试
npm test
```

### 测试助手 API

文件位置：`e2e/test-helper.ts`

```typescript
import { runTest } from './e2e/test-helper';

runTest(async (helper) => {
  // 导航
  await helper.navigate('http://localhost:5173');
  
  // 截图
  await helper.screenshot('homepage');
  
  // 交互
  await helper.click('button');
  await helper.type('input[name="title"]', '测试标题');
  await helper.press('Enter');
  
  // 等待
  await helper.waitFor('.success-message');
  await helper.sleep(1000);
  
  // 执行 JS
  const title = await helper.eval<string>('document.title');
});
```

### 已配置的测试环境

| 组件 | 版本 | 路径 |
|------|------|------|
| Playwright | latest | `e2e/` |
| Chromium | 147.0.7727.15 | `C:\Users\admin\AppData\Local\ms-playwright\chromium-1217` |
| bunwv | 0.0.5 | 全局安装 (备用) |

### 测试文件位置

- 测试代码：`e2e/*.spec.ts`
- 测试截图：`e2e/screenshots/`
- 测试报告：`playwright-report/`
- 配置：`playwright.config.ts`

---

## 📋 项目背景

**StoryForge (草苔)** - AI 辅助小说创作桌面应用

- **版本**: v5.6.4
- **GitHub**: https://github.com/91zgaoge/StoryForge
- **技术栈**: Tauri 2.4 + Rust 1.94 + React 18 + TypeScript 5.8 + SQLite + Vitest

### 双界面架构

| 界面 | 用途 | URL |
|------|------|-----|
| 幕前 (Frontstage) | 沉浸式写作 | `/frontstage.html` |
| 幕后 (Backstage) | 工作室管理 | `/index.html` |

---

## 🎨 编码风格

### Rust 后端

- 使用 `snake_case` 命名
- 错误处理使用 `Result<T, E>`
- 异步函数使用 `async/await`
- 数据库使用 `rusqlite` + `r2d2` 连接池

### TypeScript 前端

- 使用 `camelCase` 命名
- 组件使用函数式组件 + Hooks
- 状态管理使用 Zustand
- API 调用使用 TanStack Query

### 提交信息格式

```
<type>: <subject>

<body>

type:
  feat: 新功能
  fix: 修复
  docs: 文档
  style: 格式
  refactor: 重构
  test: 测试
  chore: 构建
```

---

## 🔧 开发命令

```bash
# 启动前端开发服务器
cd src-frontend && npm run dev

# 启动 Tauri 应用
cd src-tauri && cargo tauri dev

# 构建生产版本
cd src-tauri && cargo tauri build

# 运行测试
npm test
```

---

## 📚 重要文档

- [ARCHITECTURE.md](./ARCHITECTURE.md) - 架构设计
- [TESTING.md](./TESTING.md) - 测试文档
- [CHANGELOG.md](./CHANGELOG.md) - 更新日志
- [ROADMAP.md](./ROADMAP.md) - 开发路线

---

### 最近完成的功能

- **v5.6.4 Tauri v2 IPC `rename_all = "snake_case"` 根本修复** (2026-05-08) — 彻底消灭 camelCase↔snake_case 参数不匹配导致的 IPC 静默失败。根因：Tauri v2 默认将 Rust snake_case 自动转换为 camelCase 传给 JS，前端改为 snake_case 后未同步禁用转换，参数全部静默丢弃。修复：157 个 `#[tauri::command]` 全部添加 `rename_all = "snake_case"`（`lib.rs` 63 + `commands_v3.rs` 92 + `subscription/commands.rs` 2）。`cargo check` 零错误，`cargo test` 217/217 通过。
- **v5.6.3 IPC 参数一致性全面修复 + Bootstrap 序列化修复** (2026-05-08) — 修复幕后界面功能不可用的根本原因。Bootstrap 进度卡死：`CharacterElement`/`SceneElement` 添加 `#[serde(default)]` 容错 LLM 省略字段；`BootstrapProgressEvent` 新增 `status` 字段。IPC 参数全面审计：修复 7 处 camelCase↔snake_case 不匹配（前端传参修复）。后端命令参数补全：`run_creation_workflow` mode 映射、`update_story` genre、`create_character`/`update_character` 扩展字段。`cargo check` 零错误，`npm run build` 通过。
- **v5.6.2 设计-实现对齐全面修复 v5** (2026-05-08) — 全面检视并修复 5 项设计-实现差距
  - **前端缓存同步精确化**: `writingStyle` case 同时刷新 `writing_style` 缓存（修复只刷新 `world_building` 的遗漏）；`chapterUpdated` 补充 `['chapters', storyId]` 精确刷新
  - **update_scene 向量索引闭环**: `update_scene` 内联 Ingest 补充 `embed_text_async` → `VectorRecord` → `add_record`，Scene 内容变更后语义搜索可检索；`VECTOR_STORE`/`embeddings` 可见性提升为 `pub(crate)`
  - **storySelected 关联数据自动刷新**: `case 'storySelected'` 补充 8 项关联数据 `invalidateQueries`，消除切换故事时的时序依赖
  - **dataRefresh 完整覆盖**: 补充 `knowledgeGraph`/`characterRelationships` 单独 case
  - **编译优化**: 5 处 dead_code 警告清理，warnings 113→109
  - **编译**: `cargo check` 零错误，`npm run build` 通过

- **v5.6.1 设计-实现对齐全面修复 v4** (2026-05-08) — 全面检视并修复 8 项设计-实现差距
  - **幕前幕后自动关联补全**: `sceneCreated`/`sceneDeleted` 同步刷新 `chapters` 缓存，消除场景-章节关联状态滞后
  - **自适应学习真实反馈**: `record_feedback` 返回 `Vec<LearningPoint>`，同步挖掘真实偏好；前端使用返回结果替代硬编码 mock
  - **前端缓存同步完整覆盖**: `useSyncStore` 新增 `writingStyle`/`storyOutlines`/`foreshadowings` case，所有数据类型修改后自动刷新
  - **Pending vector SQLite 持久化**: Migration 42 创建表，替代 JSON 文件持久化
  - **Workflow 幂等性**: `schedule_execution` 入队前检查 queue/running，防止重复执行
  - **编译**: `cargo check` 零错误，`cargo test` 217/217 通过，`npm run build` 通过

- **v5.5.0 设计-实现对齐全面修复** (2026-05-07) — 全面检视并修复 10 项设计-实现差距
  - **幕前幕后自动关联补全**: `create_world_building`/`update_world_building` 正确发射 `WorldBuildingUpdated` 同步事件；`ChapterRepository::delete` 添加事务清理 `scenes.chapter_id` 外键；`characterDeleted` 按 `storyId` 精准失效缓存
  - **后台自动化闭环**: `auto_ingest_chapter` 成功后写入 LanceDB 向量存储（`embed_text_async` → `VectorRecord` → `add_record`），语义搜索可检索最新写作内容；WorkflowEngine 支持数据库持久化（Migration 41 + `with_pool` + 自动 save/load）；能力进化反馈环闭合（`evolve_capability_descriptions` 自动保存 + `build_default_registry` 加载进化描述 + PlanExecutor 后台触发）
  - **技术债务清理**: 移除 `src-core` 幽灵 crate（54 文件零引用）；同步 `FEATURES.md`/`ROADMAP.md`/`ARCHITECTURE.md` 版本号至 v5.4.1
  - **编译**: `cargo check` 零错误，`cargo test` 217/217 通过，`npm run build` 通过

- **v5.4.1 Bootstrap 编辑器内容丢失修复** (2026-05-07) — 修复创世流程"小说已创建但编辑器无文字"的竞态条件问题
  - `FrontstageEvent::ChapterSwitch` 新增 `content` 字段，后端直接传递生成内容
  - 前端优先使用事件中的 `payload.content`，绕过 DB 查询竞态
  - `chaptersRef` 为空时自动重新查询数据库
  - `final_content` 兜底机制
  - `loadStories` 在生成期间禁止自动 `selectStory`
  - **编译**: `cargo check` 零错误，`npm run build` 通过

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [91zgaoge/StoryForge](https://github.com/91zgaoge/StoryForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
