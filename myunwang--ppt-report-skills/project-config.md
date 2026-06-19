---
trigger: always_on
description: 汇报 PPT 生成器 / 数据报告演示稿生成工具——把数据汇报做成 16:9 网页版 PPT（设计稿 1600×900，浏览器自适应缩放），可全屏播放、一键导出 PDF。分页拆分源码 + 数据物理分离 + 多主题可切换 + ECharts 图表。当用户要"做月度/季度汇报"、"做工作汇报 PPT"、"把数据做成网页/PDF 形式的演示稿"，或希望复用一套"每页一个文件、数据单独存放、主题可换"的 PPT 模板时，**务必使用本 skill**。即使用户没说"PPT"二字，只要任务是把"多组数据 + 结论"产出为可演示/可分享的网页报告，也按本 skill 的工作流走。
---


# PPT Report Generator — 汇报 PPT 生成器（网页版 PPT 模板）

把"多组业务数据 + 阶段性结论"产出成一份 16:9 的、可在浏览器全屏播放、可一键导出 PDF 的网页版 PPT。

## 核心承诺（设计目标）

1. **每页 PPT = 三个文件**：`slides/slide-N.html` + `scripts/slide-N.js` + `styles/slide-N.css`，由 `build.py` 合成单一 HTML。改一页只动一页，**省 token、省冲突**。
2. **数据与渲染物理分离**：所有图表/表格的数据放在 `data/slide-N.{xlsx,csv,json}`，build 时自动转 JSON 注入。**日常用 Excel 改数据**，渲染代码完全不动。
3. **图表选型有规则**：每种数据形态对应一种图表类型（见 `references/chart-mapping.md`），**不要随心情画图**。默认使用 **ECharts**。
4. **信息层次有硬规则**：标签 / 大标题 / 副标题（结论）/ 区块标题 / 卡片标题 / 正文 / 注释 / 辅助文字 8 级，每级字号、字重、颜色都固定（见 `references/design-system.md`）。
5. **多主题可切换**：5 套预设主题（modern-light / dark-tech / warm-business / brand-blue / minimal-mono），通过 `data-theme` 属性切换（见 `references/themes.md`）。

## 何时启用本 skill

触发条件（任意一条命中即用）：

- 用户说："做一份月度/季度/项目汇报"、"做工作汇报"、"做数据汇报 PPT"、"把数据做成可演示的网页"
- 用户给了一份原始数据（xlsx / 多个表格 / 一段文字描述结论）并希望产出"汇报"
- 用户已有一份这种结构的项目（`src/slides/`、`build.py`、`shell.html`）想新增一页或换主题
- 用户想"把这个 PPT 沉淀下来下次复用"

## 工作流（标准步骤）

### Step 1 — 理解输入与产出

- 问清楚（如果不知道）：**几页？每页讲什么？关键数据在哪？目标读者是谁？要不要 PDF？**
- 不要直接动手画图。先把每页的 **(label, title, subtitle/结论, 主体类型)** 列出来给用户确认。

### Step 2 — 选模板（每页）

打开 `assets/slides-templates/`，按页面"主体类型"挑一个起点：

| 主体类型 | 模板 | 用法 |
|---|---|---|
| 月度交付总览（多板块 × 多卡片） | `kpi-overview.html` | 第 1 页类汇总页 |
| 两个对象左右对照（如两国数据） | `two-country.html` | KPI 行 + 双卡片 metric 表 |
| 三阶段时间线 + 多个图表 | `three-phase.html` | 时间线 + 多 chart-card |
| 多对象时间趋势 | `multi-trend.html` | 4 国 / 4 渠道 折线图 + 里程碑标记 |
| 分类条形 + 趋势折线组合 | `supply-bars.html` | 物料/品类 mini-bar + 总趋势 |
| 产业图谱 / 竞品全景 / 客户分层（一个领域/生态的完整图谱） | `landscape-map.html` | **⚠ 用前必读 [`references/landscape-skeleton.md`](references/landscape-skeleton.md) 选骨架** —— 这套三层 tier 只适合 AI/SaaS 软件分层栈；实体产业/航天/能源等用价值流横轴；银行用客户矩阵；创新药用研发管线。选错骨架 = 八股套用,会被一眼识破 |
| 2×2 战略矩阵（波士顿 / GE-McKinsey 风） | `matrix-2x2.html` | 在两个连续维度上定位 N 个对象的咨询报告标配页型。**典型场景:竞品估值定位 / BCG 业务段矩阵 / 客户分层 / 项目优先级**。纯 CSS 实现(不要用 ECharts scatter — 4 象限着色 + 标签防遮挡用 CSS 干净得多)。详见 [`references/matrix-2x2.md`](references/matrix-2x2.md) — 含 6 套经典轴组合 + 数据归一化公式 + 标签防遮挡规则 |
| 人物画像 / 拟人化能力 / 风险热图 | `human-portrait.html` | **数据驱动一次成 ⭐**。中央人体剪影 + 部位**可见色点** + 周边标签 + 引线**自动连接**。只写一段 `labels` 数据(每个标签绑定一个身体 `part`),剪影 path / 部位坐标 / 色点 / 引线全部由 `assets/scripts/silhouette.js` 的 `renderHumanPortrait()` 算出 —— 无需手画剪影 / 校准坐标 / 摆引线像素。详见 [`references/human-portrait.md`](references/human-portrait.md) |
| 价值流横轴(实体产业「价值如何沿链条流动」) | `value-chain.html` | **B 骨架 · 实体产业框架图**。5~6 个环节卡横向 ▶ 串联,每段含「核心数字 + 关键活动 + 自营/外采徽章」,色彩沿链渐深表达价值聚集。适用:商业航天 / 能源 / 制造 / 消费品。纯 CSS 全静态,零 JS。⚠ 用前先读 [`references/landscape-skeleton.md`](references/landscape-skeleton.md) 确认骨架 |

模板是骨架，**复制后改文案、改数据、改 ID** 即可。

### Step 3 — 数据放进 `src/data/slide-N/`

**这一步最重要**。**每页一个文件夹**，里面放任意多个独立数据文件，文件名（不含扩展名）即 JS 里的 key，build 时自动合并注入为 `window.__DATA_N__`。

```
src/data/slide-3/
├── kpis.xlsx      →  window.__DATA_3__.kpis   （单 sheet 自动解包为数组）
├── trend.csv      →  window.__DATA_3__.trend
└── marks.json     →  window.__DATA_3__.marks
```

**支持三种格式**，可混用：

| 格式 | 转换结果 |
|---|---|
| `.xlsx` 单 sheet | `[{col: val}, ...]` 数组（**自动解包**，省掉一层 key） |
| `.xlsx` 多 sheet | `{sheetA: [...], sheetB: [...]}` dict |
| `.csv` | `[{col: val}, ...]` 数组 |
| `.json` | 原值直接读取 |

**约定**：第 1 行 = 表头；数字自动转 number；sheet 名 / 列名 / 文件名以 `_` 开头跳过（备注用）；空行跳过。

**JS 里访问**（多数据源各自独立）：

```js
function initSlide3() {
  const D = window.__DATA_3__;
  // D.kpis  = [{label: 'DAU', value: 12.4}, ...]    ← 来自 kpis.xlsx
  // D.trend = [{week: 'W1', na: 12.1, eu: 8.4}, ...]  ← 来自 trend.csv
  // D.marks = [{x: 'W4', label: '上线'}, ...]         ← 来自 marks.json
}
```

**下次更新数据**：直接在 Excel / CSV 里改完保存，重跑 `python3 build.py` — 渲染代码完全不动。

**独立调试**：`python3 xlsx2json.py src/data/slide-3/` 可单独看该页合并后的 JSON 输出。

**向后兼容**：旧格式 `slide-N.xlsx / .csv / .json` 单文件仍支持，build 优先找目录、找不到再找单文件。详见 [`references/architecture.md`](references/architecture.md)。

### Step 4 — 选图表（看 `references/chart-mapping.md`）

不要直接选自己想画的图。根据数据形态查决策表：

- 时间序列 → 折线（带里程碑竖线 plugin）
- 对照实验前后对比 → 100% 堆叠条形
- 对象排名（Top 20） → 横向 mini-bar（自定义 div，不用 chart 库）
- 多对象单期对比 → 分组条形
- 占比/构成 → **不要饼图**，用 100% 堆叠条形或矩形树图

### Step 5 — 信息层次（看 `references/design-system.md`）

每段文字必须能回答"我是哪一级"。slide-label / slide-title / slide-subtitle 三件套是**强制顶部结构**。结论里的关键数字必须包 `<strong>` 或 `<span class="pos|neg|warn">` 高亮。

### Step 6 — 构建与预览

```
python3 build.py            # 合成最终 HTML → 输出到 dist/（目录/文件名可在 build.py 顶部 CONFIG 改）
open dist/*.html            # 浏览器打开，按 ←/→ 翻页
python3 export_pdf.py       # （可选）导出 PDF（自动找 dist/，输出 dist/*.pdf）
```

### Step 7 — 主题切换（可选）

如果用户想换风格，**不要重写 CSS**。在 `<body data-theme="dark-tech">` 上改 attribute 即可。详见 `references/themes.md`。

### Step 8 — 交付前自检（必做，别"应该没问题"就交）

构建完不等于做完。交付前**三件事一件不少**（详见 [`references/report-quality.md`](references/report-quality.md) 的 14 条铁律 + checklist）：

```
python3 check_deck.py          # 机检：字号过小 / 币种混用 / 数字缺信源 / 术语清单
python3 export_images.py dist/*.html   # 逐页截图（横屏可用 export_pdf.py）——必须肉眼过一遍
```

1. **对照 `report-quality.md` 逐条自查**：内容可信（信源 / 单一真相 / 事实观点分离）→ 表达清晰（术语解释 / 结论先行 / 分层供给）→ 版面克制（不留空白 / 字号可读 / 对齐 / 一页一事）。
2. **跑 `check_deck.py`**：把能机检的（字号、币种、信源、术语）一把过，WARN 逐条确认。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myunwang/ppt-report-skills](https://github.com/myunwang/ppt-report-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
