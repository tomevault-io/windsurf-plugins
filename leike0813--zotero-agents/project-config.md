---
trigger: always_on
description: 本项目用于开发一个 Zotero 插件，作为 ACP Agents、Skills-Runner 后端服务以及其他 API 的一个通用前端。
---

# 项目说明

本项目用于开发一个 Zotero 插件，作为 ACP Agents、Skills-Runner 后端服务以及其他 API 的一个通用前端。

# 项目特点及目标

1. 插件目标版本为 Zotero 7 + Zotero 9。
2. 插件以 zotero-plugin-template 项目为模板开发。
3. 插件采用模块化、工作流可插拔的设计理念。插件本体提供通用的 UI 界面及菜单。内部通过统一的工作流协议，由各组件分别按流程执行任务。插件本身不包含任何具体的业务逻辑。业务逻辑由用户通过“可插拔”的工作流文件或工作流包来声明和定义。
4. 插件主要以通过ACP协议调用Agent工具为目标开发，兼容旧的Skill-Runner后端服务，但也应设计为兼容其他通用的REST API后端。

# 目录结构

- 插件 TypeScript 源码：`./src`（入口 `index.ts`，通过 esbuild 构建到 `./addon`）
- 插件静态资源：`./addon`（manifest.json、bootstrap.js、prefs.js、content、locale、icons）
- 内置 Skill 定义：`./skills_builtin`、Skill 模板源码：`./skills_src`
- 内置工作流定义：`./workflows_builtin`
- OpenSpec 规格与变更记录：`./openspec`
- 参考文档/子模块：`./reference`

```shell
.
├── .github/                  # GitHub Actions workflows
├── addon/                    # Zotero 插件静态资源
│   ├── bootstrap.js          # 插件引导入口
│   ├── manifest.json         # 插件清单
│   ├── prefs.js              # 首选项默认值
│   ├── content/
│   │   ├── dashboard/        # Dashboard 页面（index.html, app.js, styles.css 等）
│   │   ├── shared/           # 静态共享资产（css、markdown renderer、theme、vendor libs）
│   │   ├── sidebar/          # 侧边栏页面（HTML/css；JS 由 src/sidebar 构建为 bundle）
│   │   ├── synthesis/        # Synthesis 工作台页面
│   │   ├── workspace/        # Assistant Workspace 页面
│   │   ├── harness/          # 只读 Harness 测试页面
│   │   ├── help-center/      # 帮助中心入口
│   │   ├── help-docs/        # 内嵌帮助文档（多语言）**自动生成，不要直接修改！**
│   │   ├── components/       # 可复用 Web 组件
│   │   ├── acp-runtime-prompts/templates/  # ACP 运行时 prompt 模板
│   │   ├── acp-skill-patches/templates/    # ACP Skill Patch 模板
│   │   └── markdown-reader/  # Markdown 附件阅读器
│   ├── locale/               # 多语言 FTL 文件（11 种语言）
│   └── bin/                  # Host Bridge CLI 预编译二进制（跨平台）
├── doc/                      # 架构文档
│   ├── components/           # 各组件设计文档（~49 篇）
│   └── synthesis-layer/      # Synthesis 层设计文档
├── reference/                # 外部参考资料（Skill-Runner, zotero-plugin-toolkit, API 指南等）
├── src/                      # TypeScript 源码
│   ├── index.ts              # 插件入口
│   ├── addon.ts              # 插件基类
│   ├── hooks.ts              # 生命周期钩子
│   ├── modules/              # 核心模块（~140 个模块文件）
│   │   ├── acp*.ts           # ACP 协议相关（connection, transport, session, skill runner, transcript 等）
│   │   ├── assistant*.ts     # Assistant 面板（model, renderer, view model, transcript）
│   │   ├── workflow*.ts      # 工作流引擎（execute, runtime, settings, menu, editor 等）
│   │   ├── skillRunner*.ts   # Skill-Runner 后端集成
│   │   ├── hostBridge*.ts    # Host Bridge 服务
│   │   ├── synthesis/        # Synthesis 子模块（~33 个文件）
│   │   ├── workflowExecution/ # 工作流执行子模块（~18 个文件）
│   │   ├── harness/          # 只读测试 Harness 子模块
│   │   └── ...               # 其他模块（backendManager, debugMode, runtimeLog, notificationHub 等）
│   ├── providers/            # 后端 Provider 实现
│   │   ├── acp/              # ACP Provider
│   │   ├── generic-http/     # 通用 HTTP Provider
│   │   ├── pass-through/     # 透传 Provider
│   │   └── skillrunner/      # Skill-Runner Provider
│   ├── backends/             # 后端注册与类型
│   ├── workflows/            # 工作流引擎核心
│   ├── utils/                # 工具函数（locale, prefs, path, fileSystem, wait, window, ztoolkit 等）
│   ├── config/               # 默认配置
│   ├── handlers/             # Handler 注册
│   ├── jobQueue/             # 任务队列
│   ├── platform/             # 平台抽象（command, env, path, subprocess）
│   ├── schemas/              # JSON Schema 定义
│   ├── shared/               # 共享前端组件与跨边界契约（citation graph, topic timeline, assistant wire/snapshot contract）
│   └── sidebar/              # 侧边栏页面 JS（ES module .js，esbuild 打包到 addon/content/sidebar/*.bundle.js；只允许 import 相对路径与 src/shared）
├── test/                     # 测试
│   ├── core/                 # 核心功能测试（~100+ 测试文件）
│   ├── node/core/            # Node.js 环境测试
│   ├── ui/                   # UI 测试
│   ├── zotero/               # Zotero 运行时测试基础设施
│   ├── helpers/              # 测试辅助工具
│   ├── fixtures/             # 测试 fixtures
│   ├── setup/                # 测试环境初始化
│   ├── mock-skillrunner/     # Mock Skill-Runner 服务
│   └── workflow-*/           # 各工作流的专项测试
├── scripts/                  # 构建与运维脚本（~50 个 .ts/.mjs）
├── skills_builtin/           # 内置 Skill 定义（~22 个 skill 目录）
│   ├── literature-analysis/
│   ├── literature-deep-reading/
│   ├── literature-explainer/
│   ├── literature-translator/
│   ├── tag-regulator/
│   ├── topic-synthesis-*/    # topic-synthesis 拆分后的多个 skill
│   └── ...                   # 其他 skills
├── skills_src/               # Skill 模板与合约源码
│   ├── topic-synthesis/      # topic-synthesis 合约、运行时、模板
│   └── literature-deep-reading/  # literature-deep-reading 合约与渲染器
├── workflows_builtin/        # 内置工作流包定义
│   ├── literature-workbench-package/  # 文献工作台工作流
│   ├── synthesis-layer/      # Synthesis 工作流
│   ├── mineru/               # MinerU 工作流
│   └── workflow-debug-probe/ # 调试探针工作流
├── openspec/                 # OpenSpec 规格与变更管理
│   ├── config.yaml
│   ├── specs/                # 规格文件（~228 个 spec）
│   └── changes/              # 变更记录（含 archive）
├── profiles/                 # Hermes Profile 发布目录
├── profiles_src/             # Hermes Profile 源文件
├── cli/                      # Zotero Bridge CLI（Rust 项目）
├── native/                   # Native 辅助程序（ACP WebSocket Bridge，Rust）
├── deprecated/               # 已废弃的旧代码（保留参考）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leike0813/zotero-agents](https://github.com/leike0813/zotero-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
