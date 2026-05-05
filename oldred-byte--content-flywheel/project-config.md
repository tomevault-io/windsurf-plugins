---
trigger: always_on
description: > 本文档记录从 2026-02-26 开始的所有功能修改，方便后续开发者快速理解项目演进。
---

# 内容飞轮 V3 - 项目修改记录

> 本文档记录从 2026-02-26 开始的所有功能修改，方便后续开发者快速理解项目演进。

---

## 2026-02-26 功能优化批次

### 1. AI 首字延迟优化

**问题描述**
用户点击"生成"按钮后，界面会卡住一段时间才显示内容，造成"假死"的感觉。

**根本原因**
- 每次生成时都要遍历所有 edges 找上游节点
- HTTP 连接建立需要时间（DNS、TLS握手等）
- 用户点击后没有即时反馈

**解决方案**
将"找上游"的过程从"点击时"提前到"连线时"：

```typescript
// store.ts - onConnect 时记录关系
onConnect: (connection) => {
  // ... 建立连线
  // 在 target 节点的 data 中记录上游节点ID
  const upstreamIds = targetNode.data.upstreamIds || [];
  updateNodeData(connection.target, {
    upstreamIds: [...upstreamIds, connection.source]
  });
}

// AIProcessorNode.tsx - 生成时直接使用
const upstreamIds: string[] = data.upstreamIds || [];
for (const sourceId of upstreamIds) {
  const sourceNode = getNode(sourceId);
  // ...
}
```

**修改文件**
- `src/store.ts`: 新增 `onConnect` 记录逻辑，`onEdgesChange` 清理逻辑
- `src/components/nodes/AIProcessorNode.tsx`: `handleGenerate` 使用 `data.upstreamIds`

**效果**
- 消除"点击后卡住"的感知
- 用户点击后立即看到"生成中"状态
- 保留所有现有功能（模板、流式输出等）

---

### 2. 提示词模板名称标准化

**修改内容**
将 5 个默认模板的名称修改为更直观的命名：

| 原名称 | 新名称 |
|--------|--------|
| AI 初稿 | 口播变初稿 |
| AI 钩子 | 稿件出钩子 |
| 小红书 | 小红书\|标题和正文 |
| 视频号 | 视频号\|标题&tag |
| 小绿书 | 口播稿转短文 |

**技术实现**
- 修改 `src/store.ts` 中的 `defaultTemplates` 定义
- 添加 `migrate` 函数自动更新用户本地缓存的模板名称

---

### 3. 模板管理系统重构

#### 3.1 允许删除默认模板
- 所有模板（包括5个默认模板）都可以删除
- 删除默认模板时提示"可恢复"
- 保留"至少保留一个模板"的限制

#### 3.2 恢复默认模板功能
- 当默认模板被删除后，显示"恢复默认模板 (X个缺失)"按钮
- 点击后自动找回被删除的默认模板
- 用户自定义模板不受影响

#### 3.3 模板拖拽排序
- 模板列表支持上下拖拽调整顺序
- 拖拽手柄显示在模板左侧（6个点图标）
- 实时保存排序结果

**技术实现**
```typescript
// store.ts 新增 action
reorderCustomTemplates: (newOrder) => {
  set({ customTemplates: newOrder });
}

// AIProcessorNode.tsx 拖拽事件处理
handleDragStart, handleDragOver, handleDrop, handleDragEnd
```

#### 3.4 模板编辑逻辑优化

**旧逻辑**
- 编辑默认模板 → 自动创建副本
- 非默认模板 → 直接保存

**新逻辑**
- 编辑任何模板 → 直接保存到原模板
- 需要创建副本 → 点击"复制模板"按钮

**界面调整**
编辑器底部按钮布局：
```
[复制模板]        [取消] [保存]
```

---

### 4. 文章转图片节点缓存优化

**问题描述**
"文章转图片"节点的设置（头像、昵称等）每次刷新后都恢复默认值，需要重新设置。

**解决方案**
将设置自动保存到节点数据中，实现持久化：

```typescript
// TextToImageNode.tsx
const [avatar, setAvatar] = useState(data.avatar || null);
const [name, setName] = useState(data.name || 'WeMD 文档');
// ... 其他状态

// 自动保存到节点
useEffect(() => {
  updateNodeData(id, {
    avatar, name, accountId, showBadge, aspectRatio, footerQuote
  });
}, [avatar, name, accountId, showBadge, aspectRatio, footerQuote]);
```

**持久化内容**
- 上传的头像（base64）
- 昵称和账号ID
- 是否显示认证标识
- 图片比例（1:1 或 3:4）
- 底部引言文字

**移除的功能**
- 移除了手动"记住信息"复选框（现在自动记住）
- 移除了 localStorage 存储逻辑（统一用节点数据）

---

## 技术架构说明

### 状态管理
- 使用 Zustand 进行全局状态管理
- 使用 `persist` 中间件实现 localStorage 持久化
- 节点数据通过 `updateNodeData` 实时同步

### 数据结构

**PromptTemplate 类型**
```typescript
export type PromptTemplate = {
  id: string;
  label: string;
  prompt: string;
  isDefault?: boolean;  // 是否为内置模板
};
```

**节点 upstreamIds**
```typescript
// 存储在节点 data 中
upstreamIds: string[];  // 上游节点ID数组
```

### 关键设计决策

1. **为什么用 upstreamIds 而不是实时遍历 edges？**
   - 减少点击生成时的计算延迟
   - 连线时关系已确定，无需每次重新计算

2. **为什么默认模板编辑不再自动创建副本？**
   - 用户反馈：需要频繁微调提示词
   - 明确区分"保存"和"复制"两个操作

3. **为什么移除"记住我"复选框？**
   - 自动保存更符合用户预期
   - 减少不必要的交互步骤

---

## 后续建议

### 可能的优化方向

1. **多设备同步**
   - 当前模板和节点数据仅存在本地 localStorage
   - 如需多设备同步，需要接入云端存储（Firebase/GitHub Gist等）

2. **模板导入导出**
   - 支持将自定义模板导出为 JSON 文件
   - 支持导入他人分享的模板

3. **模板分类**
   - 当模板数量超过 10 个时，考虑二级分类结构
   - 如：标题类、正文类、我的模板

---

## 文件修改清单

| 文件路径 | 修改次数 | 主要变更 |
|---------|---------|---------|
| `src/store.ts` | 4+ | upstreamIds 记录、模板管理 actions、migrate 函数 |
| `src/components/nodes/AIProcessorNode.tsx` | 5+ | 生成逻辑优化、模板编辑UI、拖拽排序 |
| `src/components/nodes/TextToImageNode.tsx` | 2+ | 自动缓存设置、移除 rememberMe |

---

## 用户工作流程

### 新增 AI 引擎节点后的典型操作

1. **选择模板**
   - 从下拉菜单选择预设模板（口播变初稿/稿件出钩子等）

2. **自定义模板**
   - 点击设置图标打开模板管理
   - 选中模板编辑提示词
   - 点击"保存"直接修改原模板
   - 或点击"复制模板"创建副本后修改

3. **调整模板顺序**
   - 在模板管理中拖拽调整顺序
   - 常用模板放前面

4. **删除不需要的模板**
   - 点击垃圾桶图标删除
   - 默认模板删除后可恢复

5. **生成内容**
   - 点击"生成"按钮
   - 内容流式呈现

---

---

## 2026-02-26 交互优化批次

### 1. 画布缩放功能

**需求** 用户希望通过鼠标滚轮放大/缩小整个画布

**解决方案**
在 React Flow 组件上启用内置缩放功能：

```typescript
// App.tsx
<ReactFlow
  zoomOnScroll={true}
  zoomOnPinch={true}
  zoomOnDoubleClick={true}
  minZoom={0.1}
  maxZoom={4}
/>
```

**修改文件**
- `src/App.tsx`

---

### 2. 公众号渲染节点 - 双向滚动联动

**问题描述**
- 左边编辑器滚动时，右边预览框无法同步
- 右边预览框无法主动滚动

**根本原因**
- 左边是 textarea，右边是 div，滚动行为不一致
- 预览框卡片有阴影造成视觉分层

**解决方案**

**阶段 1：修复联动逻辑**
```typescript
// 使用比例同步算法
const syncScroll = (source, target) => {
  const sourceScrollable = source.scrollHeight - source.clientHeight;
  const targetScrollable = target.scrollHeight - target.clientHeight;

  if (sourceScrollable > 0 && targetScrollable > 0) {
    const ratio = source.scrollTop / sourceScrollable;
    target.scrollTop = ratio * targetScrollable;
  }
};
```

**阶段 2：修复视觉分层**
- 移除预览卡片的 `shadow-sm` 和 `rounded-lg`
- 改为纯白色背景，融入右侧容器

**阶段 3：修复 macOS 滚动条**
- 使用内联样式 `overflowY: 'scroll'` 强制显示滚动条
- 覆盖 macOS 系统默认的 overlay 滚动条

**修改文件**
- `src/components/nodes/WechatRendererNode.tsx`
- `src/index.css`

---

### 3. 全局滚动条样式统一

**问题** 所有节点的滚动条太细，macOS 上难以触碰

**解决方案**
```css
/* 强制所有可滚动区域显示滚动条 */
.custom-scrollbar,
.overflow-y-scroll,
.overflow-y-auto {
  scrollbar-gutter: stable;
  scrollbar-width: auto !important;
  overflow-y: scroll !important;
}

/* Webkit 滚动条样式 */
::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

::-webkit-scrollbar-track {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oldred-byte/content-flywheel](https://github.com/oldred-byte/content-flywheel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
