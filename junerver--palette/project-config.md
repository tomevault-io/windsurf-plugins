---
trigger: always_on
description: Palette 是一个 Compose Multiplatform 组件库，支持 Android、Desktop (JVM)、iOS 平台，组件库（`:palette`）与示例应用（`:app`）均启用了 wasmJs 目标，因此 Web (Wasm) 端可运行同一套示例。示例应用目前覆盖 Android、Desktop、Web 三端，共享 `commonMain` 中同一套代码。
---

# Repository Guidelines

## 项目概述

Palette 是一个 Compose Multiplatform 组件库，支持 Android、Desktop (JVM)、iOS 平台，组件库（`:palette`）与示例应用（`:app`）均启用了 wasmJs 目标，因此 Web (Wasm) 端可运行同一套示例。示例应用目前覆盖 Android、Desktop、Web 三端，共享 `commonMain` 中同一套代码。

## 项目结构

```
Palette/
├── palette/                    # 核心组件库模块（发布产物）
│   └── src/
│       ├── commonMain/         # 跨平台共享代码
│       │   └── kotlin/xyz/junerver/compose/palette/
│       │       ├── core/           # 核心模块
│       │       │   ├── tokens/     # 设计令牌 (PaletteColors/ SemanticColors/Shapes/Spacing/
│       │       │   │               #   Typography/Elevation/Motion/Opacity/ControlTokens/
│       │       │   │               #   ComponentThemes, FormTokens)
│       │       │   ├── theme/      # 主题 (PaletteTheme, PaletteMaterialTheme)
│       │       │   ├── spec/       # 组件规约 (ComponentSpec, ComponentInteraction)
│       │       │   ├── i18n/       # 国际化 (PaletteStrings)
│       │       │   └── util/       # 工具类 (ModifierExtensions, PaletteDefaults)
│       │       ├── foundation/     # 基础组件 (border/BorderContainer, layout/CenterVerticallyRow)
│       │       ├── components/     # UI 组件（90+，按功能分目录，分类见下文）
│       │       │   └── chart/      # 图表子系统 PChart（多文件架构，见下文）
│       │       └── Palette.kt      # 统一导出文件（typealias）
│       ├── commonJvmAndroid/   # JVM+Android 共享
│       ├── androidMain/        # Android 特定实现
│       ├── desktopMain/        # Desktop 特定实现
│       ├── iosMain/            # iOS 特定实现 (x64/arm64/simulatorArm64)
│       ├── wasmJsMain/         # Web (Wasm) 特定实现
│       ├── commonTest/         # 跨平台单元/逻辑测试
│       ├── desktopTest/        # Desktop UI 测试（Compose 测试框架）
│       ├── desktopBenchmark/   # Desktop 逻辑基准测试（kotlinx-benchmark）
│       └── androidTest/        # Android 仪器测试
├── palette-code/               # 语法高亮子模块（Prism 对齐：C/C++/Go/Rust 等 cFamilyGrammar）
├── palette-markdown/           # Markdown 渲染子模块（含 LaTeX/TOC/YAML frontmatter）
├── palette-mermaid/            # Mermaid 渲染子模块（19 种图例，注册制 parser）
├── palette-latex/              # LaTeX 公式渲染子模块
├── app/                        # 示例应用模块（Android/Desktop/Web 三端共享 commonMain）
│   └── src/
│       ├── commonMain/         # 三端共享：App 入口 + demo/（每组件一 Demo）+ navigation/ + ui/
│       ├── commonJvmAndroid/   # JVM+Android 共享
│       ├── androidMain/        # Android 入口 + 资源（含 Noto Sans SC 字体）
│       ├── desktopMain/        # Desktop 入口
│       ├── wasmJsMain/         # Web (Wasm) 入口
│       └── test/               # JVM 单元测试
├── benchmark/                  # Android 宏基准模块（com.android.test，targetProject = :app）
└── docs-site/                  # 在线文档站点（MkDocs Material + WASM 交互预览）
```

### 组件分类（`components/`）

90+ 组件按功能分目录，与 `docs-site/docs/components/` 分类一致：

- **通用 (general)**：`button`、`text`、`image`、`tag`、`badge`、`avatar`、`statistic`、`descriptions`、`tour`、`watermark`、`barcode`、`qrcode`、`commandpalette`、`floatbutton`、`affix`、`backtop` 等
- **表单 (form)**：`checkbox`、`radio`、`switch`、`toggle`、`slider`、`rate`、`select`、`cascader`、`treeselect`、`datepicker`、`daterangepicker`、`datetimerange`、`timepicker`、`inputnumber`、`inputotp`、`textfield`、`form`、`autocomplete`、`mentions`、`colorpicker`、`transfer`、`upload`、`searchbar` 等
- **数据展示 (data-display)**：`table`、`datagrid`、`list`、`tree`、`card`、`carousel`、`collapse`、`timeline`、`progress`、`skeleton`、`tooltip`、`calendar`、`chart`、`virtuallist`、`sortable`、`infinitescroll`、`steps` 等
- **反馈 (feedback)**：`alert`、`dialog`、`drawer`、`message`、`notification`、`toast`、`popup`、`popover`、`popconfirm`、`contextmenu`、`empty`、`result`、`loading` 等
- **导航 (navigation)**：`menu`、`breadcrumb`、`pagination`、`tabs`、`bottomnavigation`、`pageheader` 等
- **布局 (layout)**：`grid`、`space`、`container`、`scaffold`、`screen`、`toolbar` 等
- **内容渲染 (feature)**：`markdown`、`mermaid`、`code`、`latex`（核心模块内置包装，重度实现拆分到对应 `palette-*` 子模块）

### 图表子系统（`components/chart/`）

`PChart` 为零第三方依赖、Compose 原生 Canvas 自渲染的可扩展图表，采用「数据 + 渲染器注册制」：

- `Chart.kt` — `PChart` 组件入口，接入 `awaitPointerEventScope`（mouse hover + touch drag）驱动 tooltip/高亮；含图例点击切换显隐、dataZoom 缩放与图表联动（`controlledZoomRange`/`onZoomChange`）
- `ChartModels.kt` — `ChartSpec`（`Pie`/`Bar`/`Line`/`Scatter`/`Radar`）、`ChartSeries`（含 `yAxisIndex` 双轴绑定）、`ChartOptions`（含 `markLines`/`dataZoom`/`showTooltip`）、`MarkLine`/`DataZoom` 模型
- `ChartLogic.kt` — 纯函数：刻度/缩放（`niceTicks`/`evenTickFractions`）、范围（`deriveYRange`/`deriveDualYRanges`）、命中检测（`hitTestPoint`/`hitTestPie`/`hitTestScatter`）、`applyZoomSlice` 切片、`computeZoom` 缩放数学、`scatterPairs`/`radarAxisAngle`/`radarVertex`
- `ChartRenderer.kt` — 渲染器分发；`BarChartRenderer`/`LineChartRenderer`/`PieChartRenderer`/`ScatterChartRenderer`/`RadarChartRenderer` 分类型绘制
- `ChartAxisRenderer.kt` — 轴/网格/双 Y 轴刻度对齐；`ChartTooltip.kt` — `ChartTooltipOverlay` + 悬浮态；`ChartAnimation.kt` — 入场动画（spring 0→1）；`DataZoom.kt` — 缩放滑块（绝对位移追踪，跟手）
- `ChartDefaults.kt` — 默认值 + `PaletteChartTokens`（含 `categoricalColors` 色板、tooltip/legend/highlight 样式，全部从语义 token 派生）

**当前能力**：柱状（分组/堆叠/横向）、折线（平滑/面积/数据点）、饼图（扇形/donut/百分比）、散点图、雷达图；tooltip 命中看数（mouse hover + touch drag）、图例点击切换显隐、入场动画、markLine 标注（平均线/目标线）、双 Y 轴、dataZoom 数据缩放、图表联动。

## 构建命令

**Windows 环境下 agent 执行 Gradle 命令时必须使用 `.\gradlew.bat`，禁止使用 `./gradlew`（会导致进程卡住）。**

```bash
# 核心构建

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junerver/Palette](https://github.com/junerver/Palette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
