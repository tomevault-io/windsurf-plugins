---
trigger: always_on
description: Create beautifully designed travel guidebook PDFs from trip itineraries. End-to-end workflow from deep research to Playwright PDF export, featuring zero AI-generated images, Tabler Icons, inline SVG decorations, and Claude's warm parchment aesthetic. Use this skill whenever the user mentions 路书, 旅行指南, travel guidebook, 行程手册, trip planner, 自驾游攻略, itinerary book, 攻略, 出行指南, 旅行计划, 行程规划, or wants to turn trip notes into a printable guide. Also triggers for multi-day travel planning, route guides, roa
---


# Travel Guidebook Maker

从调研到成书的一站式旅游路书生成引擎——**零 AI 插图、零 emoji、纯 CDN 引入**。通过 Tabler Icons + 内联 SVG 装饰 + Claude 设计美学，打造有温度、有设计感的旅行指南 PDF。

## 设计哲学

> **Monocle 的编辑品味 + DK 的视觉叙事 + Lonely Planet 的信息深度 + Claude 的温暖美学**

这不是一本冰冷的导航手册，而是一本有温度的旅行伙伴——在羊皮纸般的暖色底上，用衬线体讲述目的地的故事，用图标系统标记功能信息，用装饰元素营造编辑出版感。

**核心理念：排版即设计。** 不依赖 AI 生图或照片，通过字体层级、功能色系统、内联 SVG 装饰、CSS 伪元素，让路书本身就具备设计感。

---

## 总体流程

```
Stage 0 环境准备 → Stage 1 需求确认
→ Stage 2 深度调研 [5 个 explore agent 并行]
→ Stage 3 路线架构
→ Stage 4 内容写作 [专职 general-purpose agent]
→ Stage 5 HTML 交付与检查 → Stage 6 PDF 导出
```

Stage 0 自动检测 Node.js + Playwright 环境；Stage 2 并行调研加速信息收集；Stage 4 专职 agent 在干净上下文中写 HTML 保证一致性；Stage 5 是 HTML 检查点——用户确认满意后再执行 Stage 6 导出 PDF。

---

# Stage 0: 环境准备

> 在路书制作开始前，确保 PDF 导出环境就绪。

按以下顺序逐项检测，任一步骤失败则停止并提示用户：

### 1. Node.js

```bash
node --version
```

- ✅ v18.x 或更高 → 继续
- ❌ 命令不存在 → 停止，提示安装：https://nodejs.org/

### 2. package.json

```bash
ls package.json
```

- ✅ 存在 → 继续
- ❌ 不存在 → `npm init -y`

### 3. Playwright

```bash
node -e "require('playwright')"
```

- ✅ 无报错 → 继续
- ❌ 报错 → `npm install playwright && npx playwright install chromium`

### 4. html2pdf.mjs

已 bundled 在 `scripts/html2pdf.mjs`，Stage 6 时复制到工作目录。

### 5. 高德地图 MCP（可选增强）

尝试调用高德 MCP 工具验证可用性：

```
调用 maps_whether("{目的地城市}")
```

- ✅ 返回天气数据 → 标记 `AMAP_AVAILABLE = true`
- ❌ 工具不存在或报错 → 标记 `AMAP_AVAILABLE = false`

> 高德 MCP 是**增强层**——不可用时路书照常生成，仅精确数据降级为 LLM 估算。

**用户未配置时的提示**（仅提一次，不阻塞流程）：

> "💡 检测到高德地图 MCP 未配置。路书仍会正常生成，但距离/交通/天气数据将使用估算值。如需精确数据，可在 MCP 配置中添加 `@amap/amap-maps-mcp-server`。"

## 环境就绪确认

所有检测通过后输出：

> "✅ 环境准备就绪（Node.js {版本} + Playwright + Chromium{高德可用时加：+ 高德地图 MCP}）。开始制作路书。"

---

# Stage 1: 需求确认

向用户确认以下参数（能从指令推断的不问，推断不了的才问）：

> **收到，准备制作旅游路书。请确认以下信息：**
>
> 1. **路线名称**：「{从指令推断}」，可以吗？
> 2. **旅行类型**：自驾游 / 徒步 / 城市漫游 / 混合？
> 3. **目标读者**：深度文化游 / 轻松休闲游 / 探险挑战游？
> 4. **总天数**：{推断天数}，对吗？
> 5. **个性化信息**：需要在封面或末尾加上个人/团队信息吗？
>
> 直接回复修改项即可，没问题的我直接开始。

**快捷模式**：用户已说明足够信息时，跳过确认直接执行。

**默认值**：旅行类型=自驾游，目标读者=深度文化游，天数=根据路线推断，个性化信息=无。

## 智能路由

需求确认后，根据用户输入决定起始 Stage：

| 用户提供的内容 | 跳转到 | 说明 |
|---------------|--------|------|
| 仅目的地/天数 | Stage 2 | 标准全流程 |
| 已有行程文件（Markdown/文本） | Stage 3 | 读取行程提取架构，跳过调研 |
| 已有行程 + "已确认/不用调研" | Stage 4 | 直接进入写作排版 |
| 已有 HTML 路书 | Stage 5 | 直接进入检查，可修改后导出 PDF |

判断依据：用户是否附带了 `@文件名` 引用、是否明确说"行程已确认"、"不需要调研"等。遇到不确定的情况，默认走完整流程，但主动问一句"需要我做调研还是直接开始写？"。

---

# Stage 2: 深度调研（并行 Sub-Agent）

旅游路书的调研需要同时覆盖**实用信息**和**文化深度**。采用 **并行 explore agent** 加速调研——每个方向一个独立 agent，同时出发，互不阻塞。

## 并行调研架构

```
主 Agent                         explore agent 池（并行）
┌──────────┐                    ┌─────────────────────────┐
│ 构造 5+1  │  ── background ──→│ research-transport       │
│ agent     │  ── background ──→│ research-attractions     │
│ prompt    │  ── background ──→│ research-food            │
│          │  ── background ──→│ research-culture         │
│          │  ── background ──→│ research-practical       │
│          │  ── background ──→│ research-spatial (高德)   │
│          │                    └─────────────────────────┘
│ 等通知    │  ←── 自动通知 ───  (各 agent 完成后)
│ 收集合并   │  ── read_agent ──→ 获取每个 agent 结果
│ 输出报告   │
└──────────┘
```

> `research-spatial` 仅在 `AMAP_AVAILABLE = true` 时启动。不可用时只启 5 个 agent。
```

### Agent 分工

使用 `task` 工具，`agent_type: "explore"`，`mode: "background"`，同时启动 5+1 个 agent：

| Agent 名 | 调研方向 | 搜索关键词示例（中英结合） |
|----------|---------|----------------------|
| `research-transport` | 交通路线 | "{起点}到{终点} 高铁/自驾 时刻 价格", "{destination} transport" |
| `research-attractions` | 核心景点 | "{目的地} 必去景点 门票 开放时间 TOP10", "{destination} attractions" |
| `research-food` | 当地美食 | "{目的地} 特色美食 推荐餐厅 美食街 人均", "{destination} local food" |
| `research-culture` | 文化历史 | "{目的地} 历史 文化 典故 民俗 方言", "{destination} history culture" |
| `research-practical` | 实用信息 | "{目的地} {月份}天气 预算 安全 注意事项", "{destination} travel tips" |

> 自驾游加第 7 个 agent `research-road`（路况、加油站、海拔变化）。

### 高德空间数据 Agent（AMAP_AVAILABLE = true 时）

第 6 个 agent `research-spatial`，专职采集高德地图数据：

```
你是空间数据采集专员。使用高德地图 MCP 工具采集以下数据。

目的地：{目的地}
已知景点列表：{从需求确认中提取的所有景点名称}

## 任务 1: 景点 POI 数据
对每个景点调用 maps_text_search("{景点名}")，提取：
- POI ID, 名称, 完整地址, 经纬度, 评分, 营业时间, 联系电话

## 任务 2: 周边发现
对每个核心景点（每天的主要景点）调用 maps_around_search：
- 关键词="餐厅|小吃", location={景点坐标}, radius=1000 → 附近餐厅 top 5
- 关键词="便利店|药店", location={景点坐标}, radius=500 → 应急设施

## 任务 3: 天气预报
maps_whether("{目的地城市名称或 adcode}")

## 任务 4: 地理编码
对所有地名调用 maps_gep 获取精确坐标，供 Stage 3 路径规划使用

## 输出格式
### 景点 POI 汇总
| 景点 | 地址 | 经纬度 | 评分 | 营业时间 |
|------|------|--------|------|---------|
| ... | ... | ... | ... | ... |

### 周边发现
#### {景点名} 周边
| 名称 | 类型 | 距离 | 评分 |
|------|------|------|------|
| ... | ... | ... | ... |

### 天气预报
| 日期 | 天气 | 温度 | 风力 |
|------|------|------|------|
| ... | ... | ... | ... |
```

> 自驾游加第 6 个 agent `research-road`（路况、加油站、海拔变化）。

### Agent Prompt 模板

每个 agent 的 prompt 遵循以下结构：

```
你是旅游调研专员，负责【{方向}】调研。

## 任务参数
- 目的地：{目的地}
- 出发地：{出发地}
- 日期：{具体日期}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geekjourneyx/travel-guidebook](https://github.com/geekjourneyx/travel-guidebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
