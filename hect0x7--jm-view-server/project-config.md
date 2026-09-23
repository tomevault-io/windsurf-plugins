---
trigger: always_on
description: > 本项目原名 `plugin-jm-server`，自 v0.2.4 起更名为 `jm-view-server`（PyPI 包名 `jm-view-server`，import 包 `jm_view_server`）。CLI 命令 `jms` 与 jmcomic 插件 key `jm_server` 保持不变。
---

# jm-view-server 项目简介

> 本项目原名 `plugin-jm-server`，自 v0.2.4 起更名为 `jm-view-server`（PyPI 包名 `jm-view-server`，import 包 `jm_view_server`）。CLI 命令 `jms` 与 jmcomic 插件 key `jm_server` 保持不变。

### 1. 项目概览
`jm-view-server` 是一个基于 Python Flask 的文件服务器，旨在为本地漫画/图片资源提供类似“禁漫天堂”的 Web 浏览体验。它不仅支持文件夹浏览，还针对图片阅读进行了深度优化。

### 2. 技术栈
- **后端**: Python 3, Flask, threading
- **核心组件**: `jmcomic` (可选依赖，用于下载功能)
- **前端**: HTML5 (Jinja2 模板), Vanilla JS, 统一设计系统 (CSS 变量 + 深浅双主题)，内置 SVG 图标（无外部图标字体依赖），本地字体 Space Grotesk
- **特殊支持**: `pylnk3` (处理 Windows 快捷方式)

### 3. 核心模块分析

#### 3.1 后端逻辑 (`src/jm_view_server/`)

- **`app.py` + `routes/` (`JmServer` 类与领域 mixin)**:
    - **路由中心**: `app.py` 只保留服务器初始化、访问校验、路由注册和运行入口；`routes/files.py`、`pages.py`、`transfer.py`、`services.py` 分别承载文件管理、页面浏览、传输下载与系统服务方法，所有 URL 和 endpoint 名继续由 `JmServer.register_routes()` 统一映射。
    - **路径参数**: Flask 已负责 URL 查询参数的百分号解码；页面路由必须原样保留解码后的文件系统路径，不得再做 HTML 实体反转义，以支持名称中本就含有 `&amp;` 等字面量片段的目录和文件。
    - **身份验证**: 简单的密码校验机制，支持 IP 白名单。
    - **模板选择 (`url_format`)**: 主链路页面无论设备都返回同一套模板（PC/移动共用，靠 CSS 响应式适配）；旧 `m_` 前缀手机版模板已移除。
    - **下载集成**: `/stream` 接口利用 Server-Sent Events (SSE) 协议实时输出漫画下载日志。
- **`files.py` (FileManager 类)**:
    - **图片处理**: 识别常用图片格式，支持对文件夹首图作为预览封面。
    - **排序逻辑**: 实现 `natural_key` 算法，确保带数字的文件夹按逻辑顺序排列（如 1, 2, 10 而非 1, 10, 2）。
    - **路径处理**: 支持 Windows 盘符识别及 `.lnk` 链接解析。
    - **快捷方式展示**: `.lnk` 保留快捷方式名称并向前端暴露执行目标；点击执行目标，重命名 / 移动 / 删除则作用于快捷方式文件本身。

#### 3.2 前端资源 (`src/jm_view_server/static/ & templates/`)

自 v0.2.4 起，主链路页面统一换皮为一套现代设计系统（详见下方“前端设计系统”），并做成**单套响应式模板**（PC 与移动端共用，取代旧版 PC/`m_` 双模板）。旧移动端模板与专属资源现已删除。

    - **统一设计系统 (`static/css/app.css` + `static/js/app.js`)**:
    - **深浅双主题**: 全部颜色走 CSS 变量（design token），`data-theme` 切换 light/dark，选择记忆到 localStorage。
    - **PWA 移动声明**: 所有声明 PWA 能力的主模板同时保留标准 `mobile-web-app-capable`、Apple 兼容声明与可直接加载的 PNG favicon，避免 Chrome 移动端只识别到废弃的厂商前缀或额外请求不存在的 `/favicon.ico`；页面加载完成后注册 `/sw.js`，继续维持 manifest 与图标的最小 PWA 缓存能力。
    - **动态 HTML 传输契约**: 浏览器声明支持 gzip 时，服务端必须压缩大于等于 1 KiB 的普通 HTML 响应，并通过 `set_data()` 按压缩后的真实字节重算 `Content-Length`，同时返回 `Content-Encoding: gzip` 与 `Vary: Accept-Encoding`。禁止仅修改正文而保留旧长度；文件列表同时包含响应式列表/网格 DOM，长路径下未压缩页面可超过 200 KiB，移动端反复刷新会放大正文截断、页面永久 `loading` 与全部 `defer` 脚本未初始化的概率。
    - **App Shell**: 桌面端左侧竖向侧栏导航（文件浏览 / 消息 / 上传 / 设置 / 退出），窄屏（≤860px）自动切换为底部 tab 栏；恢复已保存的侧栏宽度或折叠状态时禁用首次布局动画，避免页面加载后的内容区宽度漂移。
    - **内置组件**: 统一的按钮 / 卡片 / 输入框 / 徽章 / 地址 pill / toast 通知；内置 SVG 图标库（`icon(name)`），不依赖 FontAwesome。
    - `app.js` 提供图标、偏好、主题、Toast、打开目录等全局基础工具；`app-shell.js` 单独负责 `renderShell()`、侧栏宽度和折叠状态，只有使用 App Shell 的页面才加载它。
    - **浏览器偏好中心**: `app.js` 的 `JmvPrefs` 统一管理主题、文件视图、分栏操作入口、阅读模式、图片大小、护眼、自动连播和消息昵称等 localStorage 配置，并向设置页与业务页提供同一套默认值。分栏操作入口使用 `browserOperations` / `jmv-browser-operations` 布尔偏好，默认开启；首页和设置页同时监听偏好事件与 `storage` 事件，删除 key 时按定义中的 fallback 恢复默认状态。
- **换皮后的页面模板**（均引 `app.css`/`app.js`）:
    - **`login.html`**: 左右分栏登录页，几何氛围背景，密码显隐（密码框 `type=password`）。
    - **`index.html` + `index-page.css` + `index-page.js` + `column-view.css` + `column-view.js`**: 文件浏览主页；模板只保留 Jinja 数据与 DOM，页面样式和页面交互分别由 `index-page.css` / `index-page.js` 承担，通用文件操作继续由 `index.js` 提供。支持列表、网格和 Finder 风格分栏三种视图；列表与网格支持单文件夹维度的排序偏好记忆（基于 localStorage 存储，默认修改时间倒序）；列表与网格封面默认采用 `/api/thumb` 纯内存 LRU 缩略图缓存服务（零磁盘持久化，支持 304 协商缓存）。首页 CSS 必须先于脚本声明，`app.js`、`app-shell.js`、`index.js`、`column-view.js`、`common.js` 与 `index-page.js` 必须全部在 `<head>` 使用 `defer`，让移动网络并行下载并按依赖顺序在 DOM 解析后执行，禁止恢复为串行阻塞或把页面脚本放到大量缩略图之后；列表前四项封面保留原生懒加载 `src` 作为首屏兜底，其余列表封面及全部网格封面保存在 `data-thumb-src`，仅由当前视图的 `IntersectionObserver` 在接近视口时激活，分栏不得启动隐藏副本。移动列表将封面作为首列视觉，无封面的条目以及看本/更多操作保留服务端渲染的 SVG 兜底，不依赖页尾脚本才出现；图标初始化发现已有 SVG 时必须保留，且图标库返回空值时不得清空现有内容，保证刷新链路同样稳定。分栏模式通过 `/api/list_files` 增量加载目录，桌面端最多同时显示四层并在继续深入时向左推进，窄屏降级为单列导航。刷新页面时会根据当前 URL 的 `path` 从共享根重建目录链，恢复最近四列及父列选中状态。每个分栏条目只保留一个紧凑 `⋯` 入口，桌面端在悬停、选中或键盘聚焦时渐显，触屏设备保持 44px 热区；视图切换旁的“操作”开关和设置页共享 `browserOperations` 偏好，可即时隐藏整个操作区并释放文件名空间。文件夹支持进入、看本、打包下载与管理操作，文件支持下载、看本与管理操作。快捷方式进入目标路径，但重命名、移动和删除始终使用 `manage_quoted_path` 作用于快捷方式本身。分栏保留独立的目录选择、键盘方向键导航、空目录/错误状态与看本入口，视图偏好统一记忆到 `jmv-view`。原有排序、书签和隐藏表单等 DOM 契约继续保留。多选模式下点击列表行或网格卡片的普通区域可直接切换选择，链接、按钮、复选框和列宽拖拽等特定功能控件保持优先。单项与批量删除统一使用站内确认弹窗，不调用浏览器原生 `confirm()`；删除可作用于用户已浏览到的共享根外普通路径，但仍禁止删除盘符根、默认共享根本身和 Windows 关键系统目录。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hect0x7/jm-view-server](https://github.com/hect0x7/jm-view-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
