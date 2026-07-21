---
trigger: always_on
description: > **语言要求**: 本项目与用户交流请使用中文。代码标识符、命令、路径等技术术语保持原样。
---

# AGENTS.md - Vim 配置仓库指南

> **语言要求**: 本项目与用户交流请使用中文。代码标识符、命令、路径等技术术语保持原样。

## 项目概述

这是 skywind3000 的个人 Vim/Neovim 配置仓库，模块化、跨平台（Windows/Linux/macOS），同时支持 Vim (8.0+) 和 Neovim。主要使用 VimL、Lua 和 Python 编写。

**作者**: skywind3000
**许可证**: MIT
**主要语言**: VimL (.vim), Lua (.lua), Python (.py)

## 架构概览

```
init.vim                    # Vim 入口，按顺序加载 init/*.vim 各模块
  +-- init/viminit.vim      # 基础设置、核心按键映射
  +-- init/config.vim       # 各插件配置、标签页标签
  +-- init/vimmake.vim      # 构建系统、grep、ctags/cscope 集成
  +-- init/ignores.vim      # 通配符忽略模式
  +-- init/tools.vim        # 工具函数（文件浏览器、透明度、性能分析）
  +-- init/keymaps.vim      # <space>/<tab>/F 键用户快捷键（~20KB，最大的文件）
  +-- init/plugins.vim      # vim-plug 插件组定义（与 bundle.vim 对应）
  +-- init/status.vim       # 自定义状态栏
  +-- init/misc.vim         # 代码片段插入、模板辅助
  +-- init/gui.vim          # GUI/字体/主题（gvim/MacVim/Neovim-Qt）
  +-- init/menu.vim         # 基于 QuickMenu 的开发菜单
  +-- init/unix.vim         # Unix 专属：终端兼容、备份、文件类型

skywind.vim                 # 主协调文件：平台检测、asyncrun 配置、配色方案系统、
                            # QuickUI/CJK/补全设置，加载 site/opt/* 和 module#drivers

bundle.vim                  # 使用 vim-plug 的插件声明（条件分组）
neovim.lua                  # Neovim Lua 入口：设置 lazy.nvim，加载 lua/plugins/*
init-vscode.vim             # VSCode Vim 扩展的最小化按键配置
tasks.ini                   # AsyncTasks 构建/运行/调试任务定义
```

## 目录结构

```
c:\Share\vim/
|-- init/               核心初始化模块（12 个 VimL 文件）
|-- autoload/           自动加载函数库
|   |-- asclib/         公共工具库（24 个模块），提供路径、字符串、UI、Git、Python 等公共函数，
|   |                   供仓库内其他模块广泛调用
|   |-- asyncrun/       作者自写的 AsyncRun 插件，异步运行 shell 命令，
|   |                   输出到 quickfix 窗口或内置终端，支持 13 种终端后端
|   |-- asynctask.vim   模仿 VSCode 的任务配置/管理系统（INI 格式定义构建/运行任务），
|   |                   基于 AsyncRun 执行，支持项目级 .tasks 和全局 tasks.ini
|   |-- quickui/        作者自写的 Vim 界面增强库（菜单栏、列表框、输入框、上下文菜单、
|   |                   文本框、预览窗口、弹出终端、确认对话框），基于 popup/floating window
|   |-- module/         内部高级功能模块，基于 asclib 的二次封装
|   |                   （24+ 模块：cpp、go、lsp、git、project、mode 等）
|   |-- navigator/      作者自写的浮动操作面板，类似 Emacs 的 which-key，
|   |                   提供键盘驱动的层级菜单导航（7 个模块）
|   |-- quickmenu.vim   垂直弹出菜单系统
|   |-- preview.vim     窗口管理系统（带唯一 ID 追踪）
|   |-- colorexp/       配色方案浏览器和调色板查看器
|   |-- gdv/            作者自写的 Git Diff View，side-by-side 左右分屏查看 commit diff，
|   |                   支持 fugitive/flog/gv.vim/vim-plug（6 个模块）
|   |-- gptcommit/      GPT 生成 Git 提交日志（支持 ChatGPT/Ollama）
|   |-- notify/         弹窗通知系统
|   |-- tweak/          杂项增强（cherry-pick、pastebin）
|   |-- python/         Python 集成（pyvim、parser、treesitter）
|   |-- plug.vim        vim-plug 插件管理器（junegunn，第三方）
|   +-- ...             其他：snippet、svnhelp、textobj、projectile 等
|-- plugin/             启动脚本，Vim 启动时自动加载
|   |-- asyncrun.vim    AsyncRun 核心（约 67KB）
|   |-- asynctasks.vim  AsyncTasks 核心（约 71KB）
|   |-- commands.vim    自定义命令（FileSwitch、SwitchHeader 等）
|   |-- menu_init.vim   QuickUI 菜单栏设置
|   |-- menu_keys.vim   Buffer/窗口/标签导航菜单
|   |-- template.vim    文件模板系统
|   |-- textproc.vim    文本过滤器管理，通过 :TP 命令调用 site/text/ 中的外部过滤脚本
|   |-- terminal_help.vim  终端集成
|   |-- altmeta.vim     修复控制台 Vim 的 Alt/Meta 键编码
|   |-- gutentags_plus.vim GNU Global/cscope 集成
|   +-- ...             其他：highlight、localrc、preview、rtformat 等
|-- site/
|   |-- bundle/         各外部插件的配置文件（每个插件一个文件）
|   |-- opt/            可选的内置插件（calendar、taglist、vimim 等）
|   |-- snippets/       SnipMate 代码片段（按语言分）
|   |-- ultisnips/      UltiSnips 代码片段（按语言分）
|   |-- template/       新建文件模板（按语言分：c、cpp、python、go 等）
|   |-- text/           文本过滤脚本（Python/shell：格式化、转换等）
|   |-- escript/        编辑器工具脚本（导出配色、列出路径等）
|   |-- minisnip/       Minisnip 代码片段
|   |-- samples/        示例配置文件
|   |-- doc/            速查文档（bash、emacs、gdb、git、vim 等）
|   +-- specs/          agent 沟通文档（草案讨论、分析报告、模块详细说明等）
|   |   +-- reference/  内置插件参考文档（asyncrun、asynctasks、quickui、gdv、textproc）
|-- lua/                Neovim Lua 配置，与 Vim 配置共用 autoload/ 和 plugin/ 下的脚本
|   |-- core/           核心工具（ascmini.lua、loader、utils）
|   |-- config/         Lua 配置（custom、extra、packages），入口为 init.lua
|   +-- plugins/        lazy.nvim 插件规格（basic、lsp、treesitter、telescope 等）
|-- ftplugin/           文件类型专属设置（c、cpp、python、go 等）
|-- colors/             配色方案（patch/ 补丁 + quickui/ 变体）
|-- syntax/             自定义语法文件（navigator、quickmenu、taskini、python 等）
|-- cheat/              速查表（git、linux、regex、tcpdump 等）
|-- doc/                Vim 帮助文件（asyncrun、asynctasks、terminal_help、apc 等）
|-- lib/                Python 库模块（emake、gptcommit、shell 等）
|-- tools/
|   |-- bin/            命令行工具（cheat、dotenv、fasd、q-* 系列工具）
|   |-- conf/           配置文件（clang-format、coc-settings、flake8、pylint 等）
|   |-- dotenv/         环境设置（mingw、vc、watcom、clang）
|   |-- emake/          Emake 构建配置（编译器工具链 INI）
|   |-- script/         工具脚本（视频、git-credential 等）
|   |-- share/          系统引导脚本
|   |-- utils/          asynctask CLI、drop 工具
|   |-- test/           插件测试脚本
|   +-- tips/           Vim 技巧和 fortune 文本
+-- etc/                个人 bash/zsh 等 shell 配置文件
                        （bash、zsh、fish、tmux、readline、z.lua 等）
```

## 核心概念

### 初始化流程

1. 用户的 `.vimrc` 加载 `init.vim`，可选加载 `skywind.vim`
2. `init.vim` 设置 `s:home`，添加到 rtp/packpath，然后按顺序加载 `init/*.vim`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skywind3000/vim](https://github.com/skywind3000/vim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
