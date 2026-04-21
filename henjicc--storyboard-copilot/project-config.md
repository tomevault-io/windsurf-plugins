---
trigger: always_on
description: - 产品目标：基于节点画布进行图片上传、AI 生成/编辑、工具处理（裁剪/标注/分镜）。
---

# AGENTS.md

## 1. 项目目标与技术栈

- 产品目标：基于节点画布进行图片上传、AI 生成/编辑、工具处理（裁剪/标注/分镜）。
- 前端：React + TypeScript + Zustand + @xyflow/react + TailwindCSS。
- 后端：Tauri 2 + Rust（命令式接口）+ SQLite（rusqlite，WAL）。
- 关键原则：解耦、可扩展、可回归验证、自动持久化、交互性能优先。

## 2. 代码库浏览顺序

建议按以下顺序理解项目：

1. 入口与全局状态
- `src/App.tsx`
- `src/stores/projectStore.ts`
- `src/stores/canvasStore.ts`

2. 画布主流程
- `src/features/canvas/Canvas.tsx`
- `src/features/canvas/domain/canvasNodes.ts`
- `src/features/canvas/domain/nodeRegistry.ts`
- `src/features/canvas/NodeSelectionMenu.tsx`

3. 节点与覆盖层
- `src/features/canvas/nodes/*.tsx`
- `src/features/canvas/nodes/ImageEditNode.tsx`
- `src/features/canvas/nodes/GroupNode.tsx`
- `src/features/canvas/ui/SelectedNodeOverlay.tsx`
- `src/features/canvas/ui/NodeActionToolbar.tsx`
- `src/features/canvas/ui/NodeToolDialog.tsx`
- `src/features/canvas/ui/nodeControlStyles.ts`
- `src/features/canvas/ui/nodeToolbarConfig.ts`

4. 工具体系（重点）
- `src/features/canvas/tools/types.ts`
- `src/features/canvas/tools/builtInTools.ts`
- `src/features/canvas/ui/tool-editors/*`
- `src/features/canvas/application/toolProcessor.ts`

5. 模型与供应商适配
- `src/features/canvas/models/types.ts`
- `src/features/canvas/models/registry.ts`
- `src/features/canvas/models/image/*`
- `src/features/canvas/models/providers/*`

6. Tauri 命令与持久化
- `src/commands/*.ts`
- `src/commands/projectState.ts`
- `src-tauri/src/commands/*.rs`
- `src-tauri/src/commands/project_state.rs`
- `src-tauri/src/lib.rs`

## 3. 开发工作流

1. 明确变更范围
- 先界定是 UI 变更、节点行为变更、工具逻辑变更、模型适配变更，还是持久化/性能变更。

2. 沿着数据流改动
- UI 输入 -> Store -> 应用服务 -> 基础设施（命令/API）-> 持久化。
- 禁止跨层“偷改”状态；尽量只在对应层处理对应职责。

3. 小步提交与即时验证
- 每次改动后做轻量检查（见第 6 节），通过后再继续。

4. 最后做一次完整构建
- 在功能收尾或大改合并前运行完整构建。

5. 发布快捷口令
- 当用户明确说“推送更新”时，默认执行一次补丁版本发布：基于上一个 release/tag 自动递增 patch 版本号，汇总代码变动生成 Markdown 更新日志，完成版本同步、发布提交、annotated tag 与远端推送；如用户额外指定 minor/major 或自定义说明，则按用户要求覆盖默认行为。
- 自动生成的更新日志正文只保留 `## 新增`、`## 优化`、`## 修复` 等二级标题分组与对应列表项；不要额外输出 `# vx.y.z` 标题、`基于某个 tag 之后的若干提交整理` 说明或 `## 完整提交` 区块，空分组可省略。

## 4. 架构与解耦标准

### 4.1 依赖与边界

- 模块间优先依赖接口/类型，不直接依赖具体实现细节。
- 跨模块通信优先使用事件总线或明确的 service/port。
- 展示层（UI）不直接耦合基础设施层（Tauri/API 调用）；通过应用层中转。

### 4.2 单一职责

- 一个文件只做一个业务概念；无法用三句话说清职责就应拆分。
- 工具 UI、工具数据结构、工具执行逻辑应分离（已采用：editor / annotation codec / processor）。

### 4.3 文件规模控制

- 舒适区：类 <= 400 行，脚本 <= 300 行。
- 警戒线：800 行，必须评估拆分。
- 强制拆分：1000 行（纯数据定义除外）。

### 4.4 层间通信

- 使用 DTO/纯数据对象，避免双向引用。
- Store 不应直接承担重业务逻辑；业务逻辑放应用层。

### 4.6 文档边界

- 本文档定位为“技术开发规范文档”，优先记录稳定的架构约束、分层规则、扩展流程、验证标准。
- 不记录易变的具体 UI 操作步骤、临时交互文案或产品走查细节（这些应放在需求文档/设计稿/任务说明中）。
- 当实现变化仅影响交互细节而不影响技术约束时，可不更新本文档。

### 4.5 节点注册单一真相源

- 节点类型、默认数据、菜单展示、连线能力统一在 `domain/nodeRegistry.ts` 声明，不在 `Canvas.tsx` / `canvasStore.ts` 重复硬编码。
- `connectivity` 为连线能力配置源：
  - `sourceHandle` / `targetHandle`：是否具备输入输出端口。
  - `connectMenu.fromSource` / `connectMenu.fromTarget`：从输出端或输入端拉线时，是否允许出现在“创建节点菜单”。
- 菜单候选节点必须由注册表函数统一推导（如 `getConnectMenuNodeTypes`），禁止在 UI 层手写类型白名单。
- 内部衍生节点（如切割结果 `storyboardSplit`、导出节点）默认 `connectMenu` 关闭，只能由应用流程自动创建。

## 5. UI/交互规范

- 复用统一 UI 组件：`src/components/ui/primitives.tsx`。
- 风格统一使用设计变量和 token（`index.css`），避免散落硬编码样式。
- 输入框、工具条、弹窗保持与节点对齐，交互动画保持一致。
- 节点底部控制条（模型/比例/生成/导出等）尺寸样式统一从 `src/features/canvas/ui/nodeControlStyles.ts` 引用，禁止在各节点散落硬编码一套新尺寸。
- 节点工具条（NodeToolbar）位置、对齐、偏移统一从 `src/features/canvas/ui/nodeToolbarConfig.ts` 引用；禁止通过 `left/translate` 等绝对定位覆盖跟随逻辑。
- 选中覆盖层 `SelectedNodeOverlay` 只承载轻量通用覆盖能力（如工具条），节点核心业务输入区应内聚到节点组件本体（例如 `ImageEditNode`）。
- 对话框支持“打开/关闭”过渡，避免突兀闪烁。
- 明暗主题要可读，避免高饱和蓝色抢占焦点（导航图已优化为灰黑系）。
- 快捷键应避开输入态（`input/textarea/contentEditable`）避免误触。

## 6. 命令与验证

### 6.1 常用开发命令

```bash
# 前端开发
npm run dev

# Tauri 联调
npm run tauri dev

# 自动发布（默认建议配合 docs/releases/vx.y.z.md 使用）
npm run release -- patch --notes-file docs/releases/v0.1.12.md
```

### 6.2 快速检查（优先执行）

```bash
# TS 类型检查
npx tsc --noEmit

# Rust 快速检查
cd src-tauri && cargo check
```

### 6.3 收尾检查

```bash
# 前端完整构建
npm run build

# 触发一次正式发布（会同步版本、提交、打 tag、推送）
npm run release -- patch --notes-file docs/releases/v0.1.12.md
```

说明：
- 日常迭代不要求每次都完整打包，先走 `tsc --noEmit` + 关键路径手测。
- 影响打包、依赖、入口、持久化、Tauri 命令时，再执行完整构建。
- 发布说明优先落到 `docs/releases/vx.y.z.md`，再通过 `npm run release` 或“推送更新”口令触发发布。
- `docs/releases/vx.y.z.md` 的默认格式同样只保留二级标题分组和列表正文，不写额外总标题、范围说明和完整提交清单。

## 7. 性能实践

- 禁止在拖拽每一帧执行重持久化或重计算。
- 节点拖拽中不要写盘；拖拽结束再保存（项目已按该策略优化）。
- 大图片场景避免重复 `dataURL` 转换；节点渲染优先使用 `previewImageUrl`，模型/工具处理使用原图 `imageUrl`。
- 项目整量持久化（nodes/edges/history）使用防抖 + 空闲调度（idle callback）队列，避免与交互争用主线程。
- viewport 持久化走独立轻量队列与独立命令（`update_project_viewport_record`），不要回退到整项目 upsert。
- 视口更新要做归一化与阈值比较（epsilon），过滤微小抖动写入。
- 优先使用 `useMemo/useCallback` 控制重渲染；避免把大对象直接塞进依赖导致抖动。
- 画布交互优先“流畅”而非“实时全量持久化”，可使用短延迟合并保存。

## 8. 模型与工具扩展规范

### 8.1 新模型接入

- 一模型一文件，放到 `src/features/canvas/models/image/<provider>/`。
- 在模型定义中声明：
  - `displayName`
  - `providerId`
  - 支持分辨率/比例
  - 默认参数
  - 请求映射函数 `resolveRequest`

### 8.2 新工具接入

1. 在 `tools/types.ts` 声明能力（如 editor kind）。
2. 在 `tools/builtInTools.ts` 注册插件。
3. 在 `ui/tool-editors/` 新增对应编辑器。
4. 在 `application/toolProcessor.ts` 接入执行逻辑。
5. 保证产物仍走“处理后生成新节点”链路，不覆盖原节点。

### 8.3 新节点接入

1. 在 `domain/canvasNodes.ts` 增加类型与数据结构（必要时增加类型守卫）。
2. 在 `domain/nodeRegistry.ts` 注册定义：`createDefaultData`、`capabilities`、`connectivity`。
3. 在 `nodes/index.ts` 注册渲染组件。
4. 明确手动创建策略：
   - 可手动创建：配置 `connectMenu.fromSource/fromTarget`。
   - 仅流程创建：关闭 `connectMenu`，由工具/应用服务触发。
5. 如新增分组/父子节点行为，必须同步验证删除、解组、连线清理与历史快照。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henjicc/Storyboard-Copilot](https://github.com/henjicc/Storyboard-Copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
