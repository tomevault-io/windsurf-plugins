---
trigger: always_on
description: ZTools 是一个跨平台 (macOS/Windows) 应用启动器和插件平台，类似 Alfred/Raycast。
---

# CLAUDE.md

## 项目概述

ZTools 是一个跨平台 (macOS/Windows) 应用启动器和插件平台，类似 Alfred/Raycast。
技术栈：Electron 38 + Vue 3 + TypeScript + Pinia + LMDB + WebContentsView。

核心能力：拼音搜索、插件系统（UI/无界面）、剪贴板管理、超级面板、分离窗口、WebDAV 同步、MCP Server、AI 集成、ZBrowser 浏览器自动化、离线翻译、悬浮球、网页快开。

## 开发命令

```bash
pnpm dev              # 启动开发（主进程 + setting 内置插件并行热重载）
pnpm dev:main         # 仅启动主进程
pnpm dev:setting      # 仅启动 setting 内置插件开发服务器
pnpm typecheck        # 全部类型检查（node + web）
pnpm typecheck:node   # 主进程 + preload 类型检查
pnpm typecheck:web    # 渲染进程类型检查
pnpm build            # 编译源码（含类型检查和 setting 构建）
pnpm build:mac        # 打包 macOS
pnpm build:win        # 打包 Windows
pnpm build:linux      # 打包 Linux
pnpm build:unpack     # 打包但不生成安装包（调试用）
pnpm test             # 运行测试 (vitest)
pnpm test:watch       # 测试观察模式
pnpm sync-api-types   # 同步 ztools-api-types 子模块类型
```

## 项目结构

```
src/main/                          # 主进程
  index.ts                         # 应用入口（单例锁、ZPX 文件关联）
  appWatcher.ts                    # 应用目录监听（chokidar，自动检测安装/卸载）
  managers/
    windowManager.ts               # 窗口管理、全局快捷键（1133 行）
    pluginManager.ts               # 插件 WebContentsView 生命周期管理（1882 行，核心）
    clipboardManager.ts            # 剪贴板监听和历史管理（860 行）
    pluginAssemblyCoordinator.ts   # 插件装配会话状态机（加载→就绪→显示）
    proxyManager.ts                # 代理配置统一管理
  api/
    index.ts                       # API 管理器（统一初始化所有模块 + 全局快捷键处理）
    updater.ts                     # 应用更新（蓝奏云 + 独立 updater 程序替换 asar）
    shared/                        # 主程序和插件共享 API
      database.ts                  # LMDB 数据库（命名空间隔离：ZTOOLS/ 和 PLUGIN/{name}/）
      clipboard.ts                 # 剪贴板 API
      imageAnalysis.ts             # 图像分析（sharp）
    renderer/                      # 主程序渲染进程专用 API
      commands.ts                  # 指令管理（历史、固定、启动）（1099 行）
      commandMatchers.ts           # 指令匹配纯函数（便于单元测试）
      plugins.ts                   # 插件安装/删除/市场（1761 行）
      window.ts                    # 窗口控制
      settings.ts                  # 设置管理
      system.ts                    # 系统功能
      systemSettings.ts            # Windows 系统设置集成
      systemCommands.ts            # 系统内置指令执行（截图、取色等）
      sync.ts                      # WebDAV 同步 API
      webSearch.ts                 # 网页快开搜索引擎管理
      localShortcuts.ts            # 本地启动项（自定义文件/文件夹/应用快捷方式）
      aiModels.ts                  # AI 模型配置管理（OpenAI 兼容格式）
    plugin/                        # 插件专用 API
      pluginApiDispatcher.ts       # 统一 API 分发器（注册表模式）
      lifecycle.ts                 # 生命周期（onPluginEnter/Leave）
      ui.ts                        # UI 控制（setExpendHeight、hideWindow、setSubInput、list mode）
      window.ts                    # 插件窗口管理
      dialog.ts                    # 对话框
      clipboard.ts                 # 剪贴板操作
      input.ts                     # 输入模拟（paste/type/sendInputEvent）
      shell.ts                     # Shell 命令执行
      feature.ts                   # 动态功能管理
      device.ts                    # 设备信息
      http.ts                      # HTTP 请求
      redirect.ts                  # 搜索重定向
      screen.ts                    # 屏幕功能（截图）
      toast.ts                     # Toast 通知（独立透明窗口）
      tools.ts                     # 插件工具声明（供 MCP 消费）
      ai.ts                        # AI 对话 API（OpenAI 兼容，流式）
      zbrowser.ts                  # 浏览器自动化 API
      ffmpeg.ts                    # FFmpeg 路径获取
      internal.ts                  # 内置插件专用 API（更高权限）
  core/
    lmdb/                          # LMDB 数据持久化
      index.ts                     # Database 类主入口
      lmdbInstance.ts              # 单例实例
      syncApi.ts / promiseApi.ts   # 同步/异步 API
    sync/                          # WebDAV 同步引擎
      syncEngine.ts                # 同步核心
      webdavClient.ts              # WebDAV 客户端
      pluginSyncWatcher.ts         # 插件目录变更监听（标记脏数据）
      pluginHasher.ts              # 插件目录哈希计算
    zbrowser/                      # 浏览器自动化
      zbrowserManager.ts           # 窗口池管理（每插件独立 Session）
      zbrowserExecutor.ts          # 操作队列执行器
      devices.ts / types.ts        # 设备预设和类型
    commandScanner/                # 指令扫描（macOS: .app plist / Windows: .lnk 开始菜单）
    commandLauncher/               # 指令启动
    native/index.ts                # 跨平台原生模块（C++）：ClipboardMonitor、WindowMonitor、WindowManager、ScreenCapture、ColorPicker、MouseMonitor
    systemSettings/                # Windows 系统设置（ms-settings URI）
    superPanelManager.ts           # 超级面板（鼠标中键/长按右键触发，智能识别剪贴板内容）
    floatingBallManager.ts         # 悬浮球（置顶小圆球，单击/双击触发）
    translationManager.ts          # 离线翻译（Bergamot WASM + Firefox 翻译模型）
    mcpServer.ts                   # MCP JSON-RPC 服务（暴露插件工具）
    httpServer.ts                  # HTTP API 服务（外部调用 ZTools 功能）
    ffmpeg.ts                      # FFmpeg 下载管理（按平台从 GitHub 下载）
    doubleTapManager.ts            # 双击修饰键检测（uiohook-napi）
    detachedWindowManager.ts       # 分离窗口管理（插件独立为窗口）
    pluginWindowManager.ts         # 插件创建的独立窗口管理
    internalPlugins.ts             # 内置插件定义
    internalPluginLoader.ts        # 内置插件加载器
    internalPluginServer.ts        # 内置插件开发服务器
    iconProtocol.ts                # ztools-icon:// 协议（图标加载）
    screenCapture.ts               # 屏幕截图
    logCollector.ts                # 日志收集
    globalStyles.ts                # 全局滚动条样式注入
  common/
    constants.ts                   # 常量

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZToolsCenter/ZTools](https://github.com/ZToolsCenter/ZTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
