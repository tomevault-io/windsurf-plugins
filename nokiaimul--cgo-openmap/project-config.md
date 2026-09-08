---
trigger: always_on
description: > **适用对象**：各类 AI Coding Agent（包括 DeepSeek-V3/R1 驱动的 Agent、Antigravity、Claude Code、Cursor、Cline、Roo Code 等）。
---

# AGENTS.md - CGo OpenMap AI Agent 快速上手指南

> **适用对象**：各类 AI Coding Agent（包括 DeepSeek-V3/R1 驱动的 Agent、Antigravity、Claude Code、Cursor、Cline、Roo Code 等）。
> **核心目标**：帮助 AI Agent 快速理解项目架构、遵循设计原则、高效准确地执行城市移植、线路/站点增改、样式定制及 Bug 修复任务。

---

## 1. 项目概述与技术栈

**CGo OpenMap** 是一款现代、轻量、高扩展性的开源城市轨道交通交互线路图引擎。

- **核心技术栈**：
  - **结构与渲染**：HTML5 + 原生 SVG 矢量渲染 + CSS3（基于 CSS 变量驱动的主题系统）
  - **核心逻辑**：纯原生 JavaScript (ES6+)，**零前端构建工具与打包依赖**（无需 Vite/Webpack/Node.js 构建步骤）
  - **组件体系**：原生 Web Components (`core/cgo-ui.js`，包含 `<cgo-icon>` 等自定义元素)
  - **离线与 PWA**：原生 Service Worker (`sw.js`) 与 `manifest.json`
- **运行方式**：纯静态 Web 资源，通过任意静态 HTTP 服务器（如 VS Code Live Server、`npx serve .`、`python3 -m http.server`）即可直接在浏览器中运行。

---

## 2. 核心架构最高铁律（严禁违反）

### 🚨 铁律一：核心引擎与城市业务数据彻底解耦
1. **`core/` 目录为多城市通用引擎**：
   - 负责 SVG 绘制、视口缩放漫游、手势处理、全局搜索、图例调度、主题切换、图卡弹窗等通用交互。
   - **严禁**在 `core/` 下的任何脚本中硬编码特定城市的车站 ID（如 `M101`）、特定线路名称（如 `1号线`）、特定颜色或特定城市的私有业务逻辑。
2. **`city/` 目录为城市业务数据层**：
   - 所有特定城市（如北京 `city/beijing/`、沈阳 `city/shenyang/`、上海 `city/shanghai/` 等）的车站坐标、线路走向、站距、图例结构、时刻表，**必须且只能**存放在 `city/{city_id}/` 目录下。
   - 所有新城市必须通过 `city/data.js` 的 `CITY_REGISTRY` 进行注册。

### 🚨 铁律二：零重型依赖与单文件纯粹性
- 项目面向轻量、开箱即用与跨平台部署，**严禁引入** React/Vue 等重型框架或需要额外编译器的依赖包。
- 新增功能需遵循原生 Web 标准（Vanilla JS, Web Components, Standard DOM/SVG APIs）。

### 🚨 铁律三：严禁破坏暗色/亮色主题与多端适配
- 所有颜色必须优先使用 `css/cgo_clr.css` 和 `css/style.css` 中定义的 CSS 变量（如 `var(--theme-bg)`, `var(--text-color)` 等）。
- 任何 UI 变更必须同时适配桌面端（鼠标滚轮、悬浮、拖拽）与移动触控端（多点手势捏合缩放、触控拖拽）。

### 🚨 铁律四：代码或数据修改必须同步更新 Service Worker
- 本项目基于原生 Service Worker（`sw.js`）实现离线预缓存与性能加速。
- **任何新增文件、修改车站/线路数据或核心引擎逻辑后，必须同步更新 `sw.js` 中的 `CACHE_NAME` 缓存版本号**（新增文件还须同步登记至 `ASSETS_TO_CACHE` 数组），**否则更改将无法生效**。
- 💡 **排错第一准则**：在开发与调试过程中，**若出现“无论怎么修改代码/数据，页面表现都毫无变化、怎么改都不起作用”的情况，请务必首先思考是否是 Service Worker 强缓存导致的可能性！**

---

## 3. 项目目录结构速查

```text
openmap/
├── index.html                  # 欢迎首页门户 (展示标题欢迎、已注册城市动态排序、主理人名录与导航)
├── main.html                   # 线路图核心交互画布 (包含基础DOM、SVG生成与多城市业务脚本加载)
├── LICENSE                     # 双轨开源许可协议 (GNU AGPLv3 + ODbL 1.0)
├── CONTRIBUTING.md              # 社区贡献与城市主理人指南
├── AGENTS.md                   # AI Agent 快速上手指南 (本文件)
├── QUICKSTART.md               # 初学者 AI 快速上手实操手册
├── PORTING.md                  # 城市移植详细操作指南
├── README.md                   # 开源项目说明主文档
├── readme.html                 # 网页版内置说明弹窗页面
├── privacy.html                # 隐私政策页面
├── manifest.json               # PWA 配置文件
├── sw.js                       # Service Worker 离线缓存
├── drunk/                      # Drunk 线路图智能转换系统 (早期测试版，仅供测试使用)
│   ├── index.html              # Drunk 沉浸式暗色转换工作台页面
│   ├── css/drunk.css           # 工作台专属样式
│   └── js/                     # 转换管道与识别算法
│       ├── drunk_pipeline.js   # 交互流程调度总线 (上传/渲染/编辑/导出)
│       ├── deepseek_vision.js  # DeepSeek 视觉大模型识图引擎 (客户端直连)
│       ├── pdf_vector_extractor.js # PDF & AI 矢量图层与 XMP 色板直通解析
│       ├── city_knowledge_matcher.js # 维基百科知识库动态匹配与 Levenshtein 纠错
│       ├── ocr_align_solver.js # 智能 OCR 与 8 方向文字排版求解器
│       ├── topology_tracer.js  # 线网拓扑追踪 (分支/环线/换乘)
│       ├── openmap_codegen.js  # 标准代码生成器与 5 项核心铁律自检
│       └── drunk_logger.js     # 控制台诊断追踪日志
├── docs/                       # 架构设计与二次开发文档
│   └── STATION_MODULE_GUIDE.md # 车站信息板自定义模块开发与配置指南
├── core/                       # 核心渲染与交互引擎 (多城市通用)
│   ├── script.js               # 主引擎：SVG生成、视口矩阵变换、平滑飞跃定位、事件监听
│   ├── station-board.js        # 车站信息板调度引擎与内置标准模块注册表
│   ├── cgo-ui.js               # Web Components 组件库 (<cgo-icon> 等)
│   ├── settings.js             # 偏好设置面板逻辑 (主题、全屏、清除缓存)
│   ├── help.js                 # 帮助与关于弹窗逻辑
│   ├── notice.js               # 动态公告与消息提示
│   └── tool-theme.js           # 亮暗主题切换控制器
├── city/                       # 城市数据层 (按城市解耦)
│   ├── data.js                 # 城市注册总线 (CITY_REGISTRY) 与运行时元数据
│   ├── beijing/                # 示例城市 (北京)
│   │   ├── beijing.js          # 城市特有业务关系与模块调度配置
│   │   ├── modules/            # 城市专属特色模块 (如 beijing_cultural.js)
│   │   ├── data_stations.js    # 车站坐标、中英文名、对齐方式、类型 (dot/tsf/rdot)
│   │   ├── data_lines.js       # 线路序列、颜色、站距、分支/环线配置
│   │   ├── data_legend.js      # 图例分组与分类显示
│   │   ├── data_timetable.js   # 车站首末班车时刻数据
│   │   ├── data_notopen.js     # 在建与规划虚线走向
│   │   ├── data_virtual_transfers.js # 出站虚拟换乘/站外连通映射
│   │   ├── data_scattered.js   # 孤立/特殊连接线路段
│   │   ├── staname.csv         # 拼音缩写、多音字与旧站名搜索库
│   │   └── stacard/            # 车站详情卡片与站台结构图组件
│   └── shenyang/               # 示例城市 (沈阳，社区贡献范例)
│       ├── shenyang.js         # 城市业务逻辑 (换乘站呼出线/方城文化地标等)
│       ├── style.css           # 城市专属样式表
│       ├── data_stations.js    # 车站数据 (1~4、9、10号线等)
│       ├── data_lines.js       # 线路走向与站距配置
│       └── ...                 # 图例、卡片与检索等全套数据
├── css/                        # 样式系统
│   ├── style.css               # 地图引擎核心样式、图层排版、手势动画
│   ├── cgo_clr.css             # 线路标志色与全局主题配色变量
│   ├── cgo_element.css         # UI 基础元素样式 (按钮、输入框、徽章)
│   ├── cgo_ui.css              # CGoUI 基础样式
│   └── cgo_components.css      # 车站卡片、检索面板与弹窗样式
└── assets/                     # 静态资源
    ├── icons/                  # 应用图标、车站徽标、天气地标
    └── svg/                    # 线路数字徽标 (icon@01.svg ~ icon@57.svg 等)
```

---

## 4. 关键数据结构与规范

### 4.1 城市注册 (`city/data.js`)

在 `CITY_REGISTRY` 中注册城市基础信息：
```javascript
const CITY_REGISTRY = {
    "shanghai": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NokiaimuL/CGo-OpenMap](https://github.com/NokiaimuL/CGo-OpenMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
