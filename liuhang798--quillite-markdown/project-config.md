---
trigger: always_on
description: 本文是提供给代码代理、自动化编程助手和新维护者的项目上下文。开始修改前应完整阅读本文，再按任务范围查看相关源码。
---

# 轻阅 Markdown：AI 项目技术指南

本文是提供给代码代理、自动化编程助手和新维护者的项目上下文。开始修改前应完整阅读本文，再按任务范围查看相关源码。

## 1. 项目概览

- 项目名称：轻阅 Markdown / Quillite Markdown
- 仓库：`https://github.com/liuhang798/quillite-markdown`
- 当前版本：`2.7.5`
- 开源协议：MIT
- 产品定位：极度轻量、美观、跨平台的 Markdown 阅读与编辑工具
- 支持平台：Windows x64、macOS Universal、Linux x64
- Windows 安装包：约 12 MB
- UI 语言：简体中文、English

核心产品体验是“阅读优先、编辑顺手”：普通状态显示沉浸式阅读页面；进入编辑状态后，默认左侧实时预览、右侧 Markdown 语法高亮编辑器，可通过预览标题栏切换按钮或“更多 → 编辑布局”交换左右位置。关闭实时预览进入全宽“仅编辑”；三种布局偏好保存在 localStorage 的 `editorLayout` 中。`Ctrl/⌘ + E` 切换完整预览与编辑；仅编辑时禁止实时预览调度/光标同步渲染，取消待处理图表并释放实例，恢复分栏与显式导出必须使用最新编辑内容。

## 2. 技术栈

| 层级 | 技术 | 主要职责 |
|---|---|---|
| 桌面框架 | Wails 2.13 | Go 与 WebView 前端绑定、窗口、文件拖放、平台集成 |
| 后端 | Go 1.25 | 文件读写、最近记录、草稿、图片读取、系统操作、更新检查 |
| 前端 | 原生 HTML、CSS、JavaScript | 页面结构、交互、状态管理、双语界面 |
| 构建 | Vite 7 | 前端打包，输出到 `frontend/dist` |
| 编辑器 | CodeMirror 6 | Markdown 编辑、语法高亮、学科公式、流程图可视化画布与 22 类 Mermaid 图表生成器、撤回历史、快捷键 |
| Markdown | marked | Markdown 转 HTML |
| 科学公式 | KaTeX + mhchem | 本地渲染 LaTeX 行内/块级公式、化学式与公式编号 |
| 安全清理 | DOMPurify | 清理渲染后的 HTML |
| 代码高亮 | highlight.js | Markdown 代码块高亮 |
| Windows 安装 | NSIS | 分步安装、快捷方式、文件关联、覆盖升级 |
| CI/CD | GitHub Actions | Windows、macOS、Linux 构建及 GitHub Release 发布 |

项目没有 React、Vue、Electron、数据库或远程业务服务。不要为了小功能引入大型前端框架。

## 3. 运行时架构

```text
用户操作
  ↓
frontend/index.html + frontend/src/renderer.js
  ↓ window.quilliteMarkdown
frontend/src/main.js（统一桥接层）
  ↓ Wails 生成绑定
app.go / updates.go（Go 后端）
  ↓
本地文件系统、Windows/macOS/Linux 系统能力、轻阅 Markdown 官网服务
```

`main.go` 使用 `//go:embed all:frontend/dist` 将前端产物嵌入最终可执行文件。正常构建必须先生成 `frontend/dist`；常规 `wails build` 会按 `wails.json` 自动执行前端安装和构建步骤。

## 4. 目录与文件职责

| 路径 | 职责 |
|---|---|
| `main.go` | Wails 应用入口、窗口尺寸、单实例、拖放、macOS 菜单和平台窗口配置 |
| `app.go` | 文档、文件夹、偏好设置、最近阅读、临时草稿、本地图片和系统集成 |
| `export_docx.go` | 将前端安全渲染后的文档转换为标准 DOCX（OOXML），处理文字样式、列表、表格、代码、链接和图片 |
| `export_html.go` | 将安全渲染后的文档导出为独立 HTML，保留主题、公式、代码和图片，并过滤可执行内容 |
| `export_center.go` | 保存导出预设、检测本机 Pandoc、执行扩展格式转换及保存 PNG/JPEG 长图 |
| `export_pdf.go` | 使用本机 Edge／Chrome／Chromium 无头打印生成带标题书签的 PDF，并在引擎间自动容错 |
| `updates.go` | 官网版本库更新检查、版本比较、30 天暂停提醒 |
| `app_test.go` | 后端单元测试、版本一致性和关键业务规则回归测试 |
| `frontend/index.html` | 标题栏、侧栏、阅读页、分栏编辑器、菜单及弹窗结构 |
| `frontend/src/main.js` | Wails 后端桥接、浏览器预览降级、平台检测 |
| `frontend/src/renderer.js` | 前端状态、编辑器、Markdown 渲染、文件列表和全部交互 |
| `frontend/src/math-rendering.js` | Typora/Pandoc 风格公式分隔符解析、KaTeX/mhchem 安全渲染 |
| `frontend/src/styles.css` | 主题、布局、响应式、macOS/Windows 差异和打印样式 |
| `frontend/wailsjs/` | Wails 自动生成绑定；Go 公开方法变化后需要重新生成 |
| `build/` | 应用图标、Windows 资源及 NSIS 安装器配置 |
| `build/windows/installer/project.nsi` | 自定义 Windows 安装、升级清理、快捷方式和卸载逻辑 |
| `build/windows/installer/wails_tools.nsh` | Wails 生成文件，通常不要手工修改 |
| `packaging/linux/` | DEB、AppImage、桌面文件和 MIME 配置 |
| `.github/workflows/release.yml` | 三平台构建、产物命名、Release 更新与发布说明提取 |
| `README.md` | GitHub 默认显示的简体中文项目主页 |
| `README.en.md` | English 项目主页 |
| `README.zh-CN.md` | 旧中文版链接的兼容入口，指向默认中文主页 |
| `CHANGELOG.md` | 中英文版本升级日志，也是应用更新弹窗和 Release notes 的来源 |
| `RELEASING.md` | 版本发布操作指南 |
| `push-to-github.bat` | Windows 双击自动拉取、提交并推送源码（仅 GitHub） |

`frontend/dist`、`frontend/node_modules`、`build/bin` 是生成目录，已被 `.gitignore` 排除，不应提交。

## 5. 主要功能

### 阅读

- Markdown 渲染、代码高亮、表格、引用、列表和图片。
- 自动生成右侧本页目录，支持标题搜索、层级／平铺切换、折叠记忆和准确章节定位；目录字号和默认宽度按显示器物理短边适配。
- 正文独立一行的 `[TOC]` 在阅读页与编辑预览中生成动态可点击目录，并随标题变化刷新。
- 当前章节跟随、阅读进度、阅读时长和字数估算。
- 文档内搜索、打印、定位文件、回到顶部。
- 明暗主题和阅读字号缩放。

### 编辑

- 左侧实时预览、右侧 CodeMirror Markdown 编辑。
- Markdown 语法颜色高亮。
- `Ctrl/Cmd + B` 加粗、`Ctrl/Cmd + I` 斜体、`Ctrl/Cmd + K` 链接。
- 标题、引用、有序列表、无序列表、任务列表；可视化表格设计器支持编辑单元格、增删／拖动行列、列对齐和持久列宽，并可原位编辑已有 GFM 表格。
- 从网页或 Word 粘贴富文本时自动将剪贴板 HTML 转换为 Markdown，保留常见结构并过滤危险地址；编辑器选区右键支持复制为 Markdown 或纯文本。
- 离线英文拼写检查提供错词波浪线、美式／英式词典、右键候选纠错、本文忽略和持久个人词典；代码、URL、公式等非正文区域不参与检查，文档内容不离开本机。
- 行内代码、代码块、表格行列选择、图片选择。
- 图片选择、拖拽和粘贴默认复制到文档旁的 `assets`；可选直接登录 PicGo Cloud 在线上传，或通过本机 PicGo 服务兼容其他图床，失败均自动回退本地相对路径。
- “学科公式”工具（按基础数学、代数与函数、几何、微积分、线性代数、概率统计、物理、基础化学和化学反应分类 79 种模板，参数填写、行内/块级/编号输出、实时预览和弹窗内教程入口）、LaTeX 行内/块级公式、mhchem 化学公式和 `\tag{…}` 公式编号。
- 工具栏撤回和 `Ctrl/Cmd + Z`。
- 编辑时每 10 秒自动保存。
- `Ctrl/Cmd + S` 保存、`Ctrl/Cmd + Shift + S` 另存为。

### 文档管理

- 打开单个 Markdown 或文本文件。
- 新建 Markdown 文档后立即进入编辑。
- 打开文件夹并使用资源浏览器集中查看文档。
- 左下角和软件首页均提供“图表范例”“公式范例”“格式范例”三份内置参考文档；内容由当前 37 类图表与 79 种学科公式注册表自动生成，打开后不写入最近阅读。首页还展示完整快捷键指南，阅读页“关闭预览”可返回首页且不删除最近阅读记录。
- 打开文档后立即进入最近阅读，支持多条持久置顶、拖动排序和删除单条记录。
- 单实例：再次打开 `.md` 文件时交给已有窗口处理。
- 支持拖放文件和系统文件关联。

### 导出

- 阅读页顶部和“更多”菜单只显示一个“导出文档”入口，打开统一导出界面；不得重新加入分散的 Word／HTML／PDF 快捷按钮。
- 统一导出界面原生提供 DOCX、带样式 HTML、无样式 HTML、带标题书签 PDF 与 PNG/JPEG 长图。
- PDF 优先依次调用本机 Edge、Chrome 或 Chromium 无头打印并启用文档大纲；全部引擎不可用时前端回退系统打印，软件不捆绑浏览器。
- 页眉页脚支持 `{title}`、`{date}`、`{page}`；PDF 为重复页眉页脚，其他格式放在文档首尾。
- 导出预设保存在偏好中，最多 24 条；包含格式、页眉页脚、图片清晰度和 Pandoc 参数。
- EPUB、RTF、ODT、LaTeX、MediaWiki 与自定义格式依赖用户本机 Pandoc；软件只检测或选择可执行文件，不捆绑 Pandoc。
- Pandoc 参数通过 `exec.CommandContext` 直接传递且不经过 shell，禁止参数覆盖保存窗口确定的输出路径。

### 平台与发布

- Windows：自绘标题栏、NSIS 安装、桌面快捷方式、Markdown 文件关联。
- macOS：原生左侧窗口控制按钮、系统菜单和 Command 快捷键。
- Linux：DEB 与 AppImage。
- 启动时检查轻阅 Markdown 官网版本库。
- 更新弹窗显示官网维护的 Markdown 更新说明并打开官网下载页面。
- 可暂停自动更新提醒 30 天，手动检查不受限制。

## 6. Go 数据模型

### `Document`

- `path`：文档绝对路径。
- `name`：文件名。
- `directory`：所在目录。
- `content`：UTF-8 文本内容。
- `modifiedAt`：RFC3339Nano 修改时间。
- `size`：字节数。
- `replacedPath`：新建草稿另存成功后被替换的旧路径，供前端清理列表。

### `Preferences`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuhang798/quillite-markdown](https://github.com/liuhang798/quillite-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
