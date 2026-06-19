---
trigger: always_on
description: 本文件为各种 AGENT 工具在本仓库中工作时提供上下文指引。
---

# AGENTS.md

本文件为各种 AGENT 工具在本仓库中工作时提供上下文指引。

## 项目概述

为 Synopsys Sentaurus TCAD 工具链提供语法高亮与自动补全的 VSCode 扩展。由 **TextMate 语法**（声明式 JSON）负责高亮，**纯 JavaScript 模块**（`src/extension.js`）负责关键词自动补全，**WASM 解析器**（`web-tree-sitter`）提供 AST 级语义功能——无 TypeScript、无构建步骤、无原生二进制依赖（WASM 为纯字节码，跨平台兼容）。

仓库：https://github.com/ONEGAYI/sentaurus-syntax-highlight
参考：https://github.com/jackyu-b/sentaurus-syntax-highlight

## 开发命令

- **从 Sentaurus XML mode 文件重新生成语法文件**：
  ```bash
  python scripts/syntax/extract_keywords.py
  ```
  注意：脚本中输入/输出路径是硬编码的（`d:\pydemo\modes` 和 `d:\pydemo\sentaurus-tcad-syntax\syntaxes`），运行前需修改 `main()` 中的路径。

- **测试扩展**：使用 VSCode 的 "Extension Development Host" 启动配置（从 `.vscode/launch.json` 按 F5）
- **工作树注意事项**：`git worktree` 只复制 git 跟踪的文件，`node_modules/`（含 `web-tree-sitter`）被 `.gitignore` 排除。在新工作树中开发前**必须运行 `npm install`**，否则 WASM 解析器加载失败导致所有 Tcl 语义功能（悬停、跳转、诊断、折叠）全部失效。`display_test/` 同样被忽略，测试文件可在主仓库路径下打开，不影响功能。
  - 凡代码开发、Bug修复/调试，进入工作树操作。

- **打包扩展**：
  ```bash
  vsce package
  ```
  在项目根目录生成 `sentaurus-tcad-syntax-<version>.vsix`。

## 项目结构

```
sentaurus-syntax-highlight/
├── package.json                                ← 扩展清单：7 种语言注册、贡献点、命令
├── package.nls.json                            ← 命令面板标题英文默认值
├── package.nls.zh-cn.json                      ← 命令面板标题中文翻译
├── CHANGELOG.md                                ← 版本变更日志
├── README.md                                   ← 项目说明文档（中文）
├── README.en.md                                ← 项目说明文档（英文）
├── icon.png                                    ← 扩展图标
├── language-configuration.json                 ← 遗留配置（已被 language-configurations/ 取代）
├── .vscodeignore                               ← VSIX 打包排除规则
├── THIRD_PARTY_NOTICES.md                      ← 第三方许可证声明（marked.js MIT）
├── AGENTS.md                                   ← AI 辅助开发项目概述与指引
│
├── media/                                      ← Webview 静态资源
│   └── marked.min.js                           ← marked.js v12（Markdown 渲染，~35KB）
│
├── syntaxes/                                   ← TextMate 语法 + 函数文档数据
│   ├── sde.tmLanguage.json                     ← SDE (Scheme) 语法高亮规则
│   ├── sdevice.tmLanguage.json                 ← SDEVICE 语法高亮规则
│   ├── sprocess.tmLanguage.json                ← SPROCESS 语法高亮规则
│   ├── emw.tmLanguage.json                     ← EMW 语法高亮规则
│   ├── inspect.tmLanguage.json                 ← Inspect 语法高亮规则
│   ├── svisual.tmLanguage.json                 ← Svisual 语法高亮规则
│   ├── sdevicepar.tmLanguage.json              ← SDEVICE PAR 参数文件语法高亮规则
│   ├── all_keywords.json                       ← 全工具关键词数据库（补全 + 语法共用，7 种语言）
│   ├── tree-sitter-tcl.wasm                    ← tree-sitter-tcl WASM 字节码
│   ├── sde_function_docs.{json,zh-CN.json}     ← SDE 函数文档（中英文双语，565 API）
│   ├── scheme_function_docs.{json,zh-CN.json}  ← Scheme 内置函数文档（中英文双语，191 函数）
│   ├── sdevice_command_docs.{json,zh-CN.json}  ← SDEVICE 命令文档（中英文双语，2117 关键词）
│   ├── svisual_command_docs.{json,zh-CN.json}  ← Svisual 命令文档（中英文双语）
│   ├── inspect_command_docs.{json,zh-CN.json}  ← Inspect 命令文档（中英文双语，193 条目）
│   ├── sprocess_command_docs.{json,zh-CN.json} ← SProcess 命令文档（中英文双语，222 关键词）
│   ├── tcl_command_docs.{json,zh-CN.json}      ← Tcl 核心命令文档（中英文双语，92 命令）
│   ├── emw_command_docs.{json,zh-CN.json}      ← EMW 命令文档（中英文双语，148 条目：20 section + 128 参数）
│   ├── tcl_expr_mathfunc_docs.{json,zh-CN.json} ← Tcl expr 数学函数文档（中英文双语，31 函数）
│   └── tcl_subcommand_docs.{json,zh-CN.json} ← Tcl 子命令文档（中英文双语，11 主命令 152 子命令）
│
├── language-configurations/                    ← 语言配置（注释符号、括号匹配、缩进）
│   ├── sde.json                                ← SDE 配置：行注释 `;`
│   └── tcl.json                                ← 6 种 Tcl 工具共用：行注释 `#`
│
├── snippets/                                   ← VSCode snippet JSON（按 language id 隔离）
│   ├── sde.json                                ← SDE 代码片段
│   ├── sdevice.json                            ← SDEVICE 代码片段
│   ├── sprocess.json                           ← SPROCESS 代码片段
│   ├── emw.json                                ← EMW 代码片段
│   ├── inspect.json                            ← Inspect 代码片段
│   ├── svisual.json                            ← Svisual 代码片段
│   └── sdevicepar.json                         ← SDEVICE PAR 代码片段（22 个原生 snippet）
│
├── src/                                        ← 扩展源码（纯 CommonJS，无构建步骤）
│   ├── extension.js                            ← 入口：activate() 协调入口（~824行）
│   ├── definitions.js                          ← 用户变量补全/悬停/跳转（Scheme + Tcl）
│   ├── docs-loader.js                          ← 文档加载常量（KIND_MAP/SORT_PREFIX/DETAIL_LABEL）与格式化工具
│   ├── register-sde-providers.js               ← SDE 语言 Provider 注册（Folding/Bracket/Signature/SemanticTokens）
│   ├── register-tcl-providers.js               ← 6 种 Tcl 语言共用 Provider 注册（Folding/Bracket/SemanticTokens）
│   ├── register-completion-providers.js        ← Completion/Hover/Definition Provider 注册（7 种语言共用循环）
│   │
│   ├── commands/                               ← VSCode 命令实现
│   │   ├── expression-converter.js             ← Scheme 前缀 ↔ 中缀表达式双向转换（含 CursorTracker 光标位置感知、尖括号连字符变量语法、QuickPick 变量补全和历史模式）
│   │   ├── env-var-manager.js                  ← SWB 环境变量管理命令（批量添加/搜索删除/导出/导入）
│   │   ├── help-reader.js                      ← Webview 帮助阅读器（sentaurus.openHelp，~1022行，三栏布局+搜索+大纲+状态持久化）
│   │   └── snippet-picker.js                   ← QuickPick 代码片段命令（sentaurus.insertSnippet）
│   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ONEGAYI/sentaurus-syntax-highlight](https://github.com/ONEGAYI/sentaurus-syntax-highlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
