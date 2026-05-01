---
trigger: always_on
description: 基于 Tauri v2 + Vue 3 + TypeScript 构建的桌面阅读应用。
---

# Legado Tauri — 项目指南

基于 Tauri v2 + Vue 3 + TypeScript 构建的桌面阅读应用。

## 技术栈

| 层 | 技术 |
|---|---|
| 前端 | Vue 3 (Composition API + `<script setup>`) + TypeScript |
| UI 组件库 | Naive UI 2 |
| 代码编辑器 | Monaco Editor（`monaco-editor` + `@guolao/vue-monaco-editor`） |
| 构建 | Vite 6、vue-tsc |
| 桌面壳 | Tauri 2（Rust） |
| 包管理 | pnpm |

## 项目结构

```
src/
  style.css                        # 全局 CSS 变量 & Reset
  main.ts                          # 入口：挂载 App + 配置 Monaco Worker（离线）
  App.vue                          # 根组件，CSS Grid 整体布局
  vite-env.d.ts                    # Vite 类型声明
  assets/
    booksource-default.svg         # 书源默认图标
  data/
    extensionExamples.ts           # 扩展示例数据
  components/
    layout/
      TitleBar.vue                 # 标题栏（顶部全宽）
      SideBar.vue                  # 菜单栏（左侧，可折叠）
      TaskBar.vue                  # 任务栏（右侧内容区底部）
      MainContent.vue              # 主内容区容器
      BottomNav.vue                # 移动端底部导航
      MobileDebugFloat.vue         # 移动端调试浮窗
    BookSourceEditorModal.vue      # Monaco 书源代码编辑弹窗
    ScriptDialog.vue               # 扩展脚本对话框
    BookSourceDocs.vue             # 书源开发文档组件
    bookshelf/
      ShelfBookCard.vue            # 书架书卡组件
    explore/
      BookCard.vue                 # 书籍卡片
      BookDetailDrawer.vue         # 书籍详情抽屉
      ChapterReaderModal.vue       # 章节阅读弹窗
      ExploreHtmlRenderer.vue      # HTML 发现页 iframe 渲染
      SourceExploreSection.vue     # 单书源发现分区
      SourceSearchGroup.vue        # 搜索结果书源分组
    reader/
      ReaderTopBar.vue             # 阅读器顶栏
      ReaderBottomBar.vue          # 阅读器底栏
      ReaderSettingsPanel.vue      # 阅读设置面板
      ReaderTocPanel.vue           # 目录面板
      types.ts                     # 阅读器公共类型（ReaderBookInfo, FlipMode 等）
      composables/
        useGesture.ts              # 手势识别（滑动 / 点击区域）
        usePagination.ts           # 分页计算
        useReaderSettings.ts       # 阅读器设置状态
      modes/                       # 翻页模式组件
        ScrollMode.vue             # 滚动模式
        SlideMode.vue              # 滑动翻页
        SimulationMode.vue         # 仿真翻页
        CoverMode.vue              # 覆盖翻页
        NoneMode.vue               # 无动画翻页
        ComicMode.vue              # 漫画模式
    docs/
      BookSourceDocs.vue           # 书源文档入口
      sections.ts                  # 文档章节定义
      examples/                    # 书源 API 示例代码（*.js）
  composables/
    useAppConfig.ts                # ⭐ 应用级全局配置管理（单例，优先使用）
    useBookSource.ts               # 书源 CRUD + BookSourceMeta 接口 + 模板生成
    useBookshelf.ts                # 书架 CRUD 响应式状态（全局单例）
    useEnv.ts                      # 运行环境检测（isTauri, isMobile, platform）
    useExploreBridge.ts            # HTML 发现页 iframe ↔ Vue 双向通信桥梁
    useExtension.ts                # 扩展脚本 CRUD + ExtensionMeta 接口
    useInvoke.ts                   # Tauri invoke 超时保护封装（invokeWithTimeout）
    useNavigation.ts               # 视图导航状态（activeView, navigateToSearch）
    useScriptBridge.ts             # Boa JS ↔ Rust ↔ Vue 双向调用桥梁（全局单例）
    useSourceCapabilities.ts       # 书源能力检测（search/explore/toc/content）& 开关管理
  views/
    BookshelfView.vue              # 书架
    BookSourceView.vue             # 书源管理（已安装 / 在线仓库 / 调试）
    ExploreView.vue                # 发现页
    ExtensionsView.vue             # 扩展管理
    HistoryView.vue                # 历史记录
    LogView.vue                    # 独立日志查看器（按类型/书源过滤、HTTP 详情面板）
    SearchView.vue                 # 全局搜索（多书源并发搜索）
    SettingsView.vue               # 设置（使用 useAppConfig）
src-tauri/
  src/
    lib.rs                         # Tauri generate_handler![] 注册入口 + run() + run_cli()
    main.rs                        # Rust 入口
    app_config.rs                  # ⭐ 应用级全局配置（AppConfig 单例 + Tauri 命令 + getter）
    cli.rs                         # CLI 模式书源测试（search/info/toc/content/explore/all）
    booksource/
      mod.rs                       # BookSourceMeta 结构体、parse_meta、CRUD 命令
      engine.rs                    # Boa JS 引擎、inject_legado_api、标准书源调用命令
      builtins.rs                  # 编解码、哈希、加解密等工具函数注入
      dom.rs                       # HTML DOM 解析 & CSS 选择器（scraper，legado.dom.*）
      comic.rs                     # 漫画图片并发下载 & 本地缓存
    bookshelf/
      mod.rs                       # 书架持久化（ShelfBook CRUD + 章节目录 + 正文缓存）
    extension/
      mod.rs                       # ExtensionMeta（UserScript 格式）+ CRUD 命令
    script_bridge.rs               # dialog / REPL UI 桥接命令
    script_config.rs               # 脚本配置持久化（字符串 + 字节数组，scope 隔离）
    utils/
      mod.rs                       # 路径工具、安全校验、异步执行、任务取消注册表
    watcher.rs                     # 文件系统监听（书源 & 扩展目录变动推送事件）
  tauri.conf.json
  Cargo.toml
docs/
  booksource.md                    # 书源系统详细文档
scripts/
  probe/                           # 书源自动探针工具（详见 scripts/probe/README.md）
  tfcli/                           # TaskFlow CLI 工具
```

## 布局模型

CSS Grid 双列三行：

```
grid-template-areas:
  "title   title"
  "sidebar main"
  "sidebar taskbar"
```

CSS 自定义属性（`src/style.css`）：
- `--titlebar-h` `--taskbar-h` `--sidebar-w` `--sidebar-collapsed-w`
- `--color-*` `--space-*` `--radius-*` `--transition-*`

## 全局配置系统（app_config）

> **⭐ 优先使用**：所有可配置的应用级参数（HTTP UA、超时、UI 开关等）**必须**通过 `app_config` 管理，禁止 hardcode。

### 架构

```
Rust 后端                                     Vue 前端
┌─────────────────────────┐                  ┌──────────────────────────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegadoTeam/Legado-Tauri-Release](https://github.com/LegadoTeam/Legado-Tauri-Release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
