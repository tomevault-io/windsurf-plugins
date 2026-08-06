---
trigger: always_on
description: > 最后按源码校准：2026-07-28。
---

# AGENTS.md - Relatum / 画布项目 AI 接手指南

> 最后按源码校准：2026-07-28。
> 这份文件是给后续 AI agent 的“接手地图”，不是历史任务流水账。若本文与源码冲突，以源码为准；改动功能后，要同步更新本文对应章节。

## 0. 先读这里

- 项目显示名是 **Relatum**，仓库目录仍叫“画布”。它是本地知识画布 + 学习工作台 + 桌面壳，不是 Web SaaS。
- 维护时默认用中文与用户沟通；解释要清楚、谨慎，不要假设用户知道内部实现。
- 只改当前任务真正需要的文件。用户数据目录、示例画布、已有配置不要随手改、删、格式化。
- 手动编辑文件时使用 `apply_patch`。运行格式化、检查、构建可以用命令，但不要用脚本偷偷重写源文件。
- 这个项目没有 npm 构建链。前端是原生 HTML/CSS/JS，后端主体是 Python 标准库；桌面包才依赖 pywebview / PyInstaller / Pillow。
- 不要把这份文档继续堆成杂乱备忘录。新增功能时，把“真实行为、数据位置、入口文件、验证方式”补到对应章节。

## 1. 项目一句话

Relatum 是一个离线优先的本地学习与知识组织工具：

- `editor.html` + `assets/canvas.js` 提供无限画布、卡片/索引/代码/便签/附件、手写、连线、模板、脑图、AI 注入等核心编辑能力。
- `index.html` + 多个页面模块提供最近画布、分组、收藏、回收站、学习任务、活跃星图、速记墙、日历日记、复习池、专注钟和每日任务。
- `app.py` 是本地 HTTP 服务和所有持久化逻辑，监听 `127.0.0.1`，默认端口 `8765`，自动寻找空闲端口。
- `desktop.py` 是 pywebview / WebView2 桌面外壳，启动同一个本地服务并打开桌面窗口。

## 2. 绝对边界

### 用户数据优先

- `canvases/`、`canvases/回收站/`、`data/` 是用户数据。除非任务明确要求，不要批量改写。
- `/api/new` 的默认文件名固定为 `Untitled-YYYY-MM-DD[-N].canvas`，与界面语言无关；不要改回中文前缀，也不要自动重命名旧画布。
- 删除画布应走回收站逻辑：画布文件和同名 `.assets` 目录一起移动。
- 保存 JSON / 文本时沿用后端已有的原子写入：在目标同目录写短名、进程/线程唯一的临时文件，再 `os.replace`；不要改回固定 `.tmp`，也不要给频繁自动保存额外加同步 `fsync`。
- 打开外部文件、恢复、重命名、导入、导出都必须尊重路径授权。`app.py` 只允许 `canvases/`、回收站、最近列表里的路径、以及命令行 `--allow-dir` 传入的目录。
- `open-external` 有危险扩展黑名单，不能为了“方便打开”绕开它。

### 零依赖和离线优先

- 不要引入 npm、CDN、Electron、Tauri 或前端框架。
- 已有第三方前端库放在 `assets/vendor/`，包括 MathJax、PDF.js、Mermaid。它们是离线资产，不要把页面改成在线加载。
- AI 是唯一主动出站能力：后端用标准库 `urllib` 调 OpenAI 兼容 `/chat/completions`，默认 DeepSeek。不要新增对外开放的 HTTP 控制面。

### 协议 A / 外部打开

- 外部程序若要打开画布，只传 `.canvas` 文件路径给 `app.py` 或 `desktop.py` 的命令行参数。
- 不要新增“后台常驻远程 API”“局域网同步”“自动监听外部命令”等能力，除非用户明确要求并重新设计安全边界。

### 前端性能和视觉

- `styles.css` 很大，视觉语言是纸张、墨色、温和强调色。不要把界面改回大面积科技蓝、紫蓝渐变或重毛玻璃风格。
- 避免持续 `backdrop-filter`、大面积 blur、无限 keyframe 动画、滥用 `will-change`。这些在旧优化记录里明确踩过坑。
- 编辑器启用深色语义且背景判定为深色时，右侧完整面板、简洁样式面板与简洁节点入口统一使用高不透明度的墨绿黑表面、细亮边框和暖白选中态；这些常驻面板不使用 `backdrop-filter`，避免背景透亮发白和持续模糊开销。
- 大画布、图谱、PDF、MathJax、Mermaid 都在热路径附近。改动 `canvas.js`、`graph-engine.js`、`graph-gl.js` 前先定位最小区域。
- MathJax 与 Mermaid 都是本地按需运行时：前者只由真实公式源触发，后者只由 Mermaid fence 触发。普通文字/普通 Markdown 不应加载它们，也不要恢复全 `document.body` 的 Mermaid MutationObserver；所有生成 Markdown DOM 的入口必须显式调用对应渲染器。
- 画布节点按 id 使用常驻索引供连线热路径查询；静态连线由 Canvas 绘制，拖节点和脑图滑行期间临时切到 SVG 增量更新，收尾后一次性重建 Canvas。删除或快照重建连线时必须同步清理 SVG marker 与 `edgePathCache`。
- `prefers-reduced-motion` 已在多处使用；新增动画要考虑降级。

## 3. 源码地图

| 路径 | 责任 |
| --- | --- |
| `app.py` | 本地 HTTP 服务、路由、持久化、导入导出、AI 代理、独立复习卡片数据库、学习/日历/速记/专注数据。 |
| `desktop.py` | pywebview 桌面壳、WebView2 检测、无边框窗口、窗口状态、未保存关闭确认。 |
| `build-desktop.ps1` | PyInstaller onedir 打包，输出 `Relatum-release/Relatum.exe`。脚本保持 ASCII。 |
| `start.ps1`、`打开画布.bat` | 源码模式启动器，查找 Python 并运行 `app.py`。 |
| `index.html` | 起步页壳，书脊导航、最近画布、学习/速记/日历/复习/专注入口。 |
| `editor.html` | 画布编辑器壳，工具栏、各模式面板、读者浮层、AI 面板、图谱浮层。 |
| `trash.html` | 回收站管理页。 |
| `assets/start.js` | 起步页状态、最近/分组/收藏、页面切换、主题/背景/翻页速度。 |
| `assets/editor.js` | 编辑器页面编排：加载/保存、模式切换、模板、导出、背景、AI/图谱入口。 |
| `assets/editor-onboarding.js` | 编辑器首次使用引导：十一页 CSS 演示浮窗、翻页/重播、中英文案和真实画布四步练习。 |
| `assets/i18n.js` | 起始页与编辑器共用的界面语言层；保存语言偏好、翻译静态/动态 UI，并保护用户内容区。 |
| `assets/tooltip.js` | 全局自定义说明框层；接管静态与动态 `title`，同步中英文文案，并处理悬停/键盘焦点与视口避让。 |
| `assets/canvas.js` | 核心画布引擎，节点/边/手写/附件/批注/选择/历史/脑图/独立表格/AI 注入。 |
| `assets/ruler.js` | 画布尺子的无 DOM 几何层；负责数据归一化、坐标旋转、有限长边投影、笔迹线段捕获与节点扫掠碰撞。 |
| `assets/canvas-import.js` | `.canvas` 内容合并的无 DOM 数据层；负责结构校验、深拷贝、ID/引用重映射、附件策略与节点/墨迹联合边界偏移。 |
| `assets/node-matrix.js` | “节点矩阵”的无 DOM 数据层；负责行列与数量校验、连续编号、Tab/换行二维粘贴、统一宽度解析和居中网格布局。 |
| `assets/canvas-timer.js` | 画布倒计时/正计时的无 DOM 数据层；负责数据规范化、真实时间差、格式化、完成判定、复位与状态取反。 |
| `assets/canvas-scenes.js` | “镜头册”的无 DOM 数据层；负责固定/跟随镜头规范化、创建、更新、删除、排序和失效引用清理。 |
| `assets/canvas-taskbook.js` | “任务簿” V3 界面的无 DOM 数据层（盘面仍为 `taskbook.version:2`）；负责顶级任务与成员树规范化、独占归属、叶子进度、计时段、枝桠工作流镜像和归档完成副本。 |
| `assets/markdown-notebook.js` | “笔记坞”的无 DOM 数据层；负责多页笔记规范化、复用共享 Markdown 结构规则生成导图层级、结构统计、画布选区反向序列化和 Enter 列表续写。 |
| `assets/ai.js` | 右侧 AI 面板、聊天、上下文模式、生成预览、确认后注入。 |
| `assets/richtext.js` | 画布文字的结构化局部格式层；管理 `textMarks` / `bodyMarks`、旧内联语法迁移、编辑 DOM 与导出序列化。 |
| `assets/markdown-table.js` | Markdown 表格的无 DOM 数据层；负责解析、规范化、序列化、CSV/TSV 粘贴与正文内表格定位。 |
| `assets/markdown.js` | 零依赖 Markdown 结构层与安全渲染器；统一标题、列表/任务项、引用、围栏、公式和段落分类，提供 `renderResult()` 的 HTML + Math/Mermaid 特征结果，并保留 `render()` 兼容入口。 |
| `assets/table-editor.js` | 通用二维网格交互层；负责单元格/行列选择、增删、粘贴、对齐、源码切换与表格工作室。 |
| `assets/mermaid-renderer.js` | 统一离线 Mermaid 渲染队列。 |
| `assets/graph-engine.js` | 通用关系图引擎，Canvas2D + 可选 WebGL 几何后端。 |
| `assets/graph-gl.js` | WebGL2 实例化渲染后端，暴露 `window.GraphGL`；只画节点/边几何，文字仍走 2D/DOM。 |
| `assets/graph-view.js` | 当前画布关系图浮层。 |
| `assets/study.js` | 学习任务、看板/清单、任务归档、任务关联画布、迷你编辑器。 |
| `assets/study-graph.js` | 活跃页/学习页星图，可视化学习活动和任务结构。 |
| `assets/notes.js` | 起步页速记墙，独立便签数据、拖拽、连线、箭头、归档。 |
| `assets/start-sticky-notes.js` | 起步页跨页便签：安全空白创建、纯文本编辑、轻量拖动、键盘换色/旋转/删除。 |
| `assets/calendar.js` | 日历、日记、任务日历、自由任务便签、倒数日。 |
| `assets/countdown.html`、`assets/countdown.js` | 独立倒数日页面；事件管理、轻量翻页时钟、空状态与返回日历过渡。 |
| `assets/review.js` | 独立复习卡片页面，负责计划复习、无限随机自由复习、卡片库、卡组/标签、批量管理和评分。 |
| `assets/focus.js` | 专注钟、正计时/番茄钟、学习/每日任务绑定、音效/噪音、记录编辑。 |
| `assets/trash.js` | 回收站页面，按目标分组恢复、键盘恢复、一键清空确认。 |
| `assets/desktop-shell.js` | 前端到 pywebview 的桥接、窗口按钮、dirty 标记。 |
| `assets/styles.css` | 全局视觉系统和所有页面样式。 |
| `assets/editor-onboarding.css` | 编辑器新手引导的纸页浮窗、十一组有限播放演示、按钮反馈、深色与低动态适配。 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yamibk/Relatum-Opensource](https://github.com/yamibk/Relatum-Opensource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
