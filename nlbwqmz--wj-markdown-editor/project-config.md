---
trigger: always_on
description: 一款开源桌面端 Markdown 编辑器，支持 Windows 和 Linux。整体架构为 Electron 39 + Vue 3 + Vite 6。
---

# wj-markdown-editor 项目上下文

## 项目概述

一款开源桌面端 Markdown 编辑器，支持 Windows 和 Linux。整体架构为 Electron 39 + Vue 3 + Vite 6。

和旧版本相比，当前代码主线已经明显收口到“文档会话”模型：

- Web 端不再依赖零散的文件状态镜像，而是以 `Pinia store.documentSessionSnapshot` 作为渲染层真相。
- Electron 端通过 `documentSessionRuntime`、`windowLifecycleService`、`documentCommandService`、`documentEffectService` 统一处理打开、编辑、保存、关闭、外部变更、最近文件与资源操作。
- Renderer 与主进程之间围绕 `document.*` 命令和 `document.snapshot.changed` 事件通信，资源打开/删除也统一经 runtime 裁决。

### 当前核心特性

- 支持本地 Markdown 文件打开、编辑、另存为副本
- 支持最近文件恢复、缺失最近文件提示、单实例文件打开
- 支持本地图片、网络图片、本地附件、音频、视频插入
- 支持截图上传、隐藏窗口截图、图片上传图床（GitHub / SM.MS）
- 支持暗黑模式、代码高亮主题、预览主题、分区字体设置
- 支持精准同步滚动、滚动锚点恢复、编辑区与预览区关联高亮
- 支持公式（KaTeX）、Mermaid、GitHub Alert、自定义 Container、脚注、图片尺寸、文字颜色
- 支持预览区资源右键菜单、在资源所在目录中打开、删除当前引用、删除全部引用、按策略删除本地文件
- 支持外部文件变更监听，可配置为直接应用或弹出 diff 对比确认
- 支持全文搜索、编辑区搜索、设置页搜索定位
- 支持导出 PDF、PNG、JPEG
- 支持水印、自定义快捷键、自动保存、启动页切换

## 项目结构

```text
wj-markdown-editor/
├── docs/                              # 项目文档
├── release-notes/                     # 发布说明
├── wj-markdown-editor-web/            # Vue 3 渲染层
│   ├── src/
│   │   ├── assets/                    # 全局样式、主题、图片
│   │   ├── components/
│   │   │   ├── editor/                # 编辑器、预览、工具栏、资源右键菜单
│   │   │   └── layout/                # 主布局与其他窗口布局
│   │   ├── i18n/                      # 中英文文案
│   │   ├── router/                    # 路由定义
│   │   ├── stores/                    # Pinia，全局配置与文档快照状态
│   │   ├── util/
│   │   │   ├── channel/               # IPC 通信与窗口事件桥接
│   │   │   ├── document-session/      # Renderer 侧文档快照、命令与激活策略
│   │   │   ├── editor/                # 编辑器行为、滚动、补全、资源操作
│   │   │   ├── guide/                 # 引导页文案
│   │   │   └── markdown-it/           # Markdown 解析扩展
│   │   ├── views/                     # 编辑、预览、设置、导出、关于、引导页
│   │   └── main.js
│   ├── vite.config.js
│   └── package.json
└── wj-markdown-editor-electron/       # Electron 主进程与桌面能力
    ├── src/
    │   ├── data/                      # 配置、最近文件、默认配置
    │   ├── util/
    │   │   ├── channel/               # IPC 主进程入口
    │   │   ├── document-session/      # 文档运行时、命令、effect、窗口生命周期
    │   │   ├── win/                   # 设置、导出、引导、关于、截图窗口能力
    │   │   └── *.js / *.test.js       # 资源、协议、更新、上传、监听等工具
    │   ├── main.js
    │   └── preload.js
    └── package.json
```

## 最新主线架构

### Web 端状态与路由

- 路由定义位于 `wj-markdown-editor-web/src/router/index.js`
- 主路由结构：
  - `/#/editor` 编辑页
  - `/#/preview` 预览页
  - `/#/setting` 设置页
  - `/#/export` 导出页
  - `/#/about` 关于页
  - `/#/guide` 引导页
- `wj-markdown-editor-web/src/stores/counter.js` 是当前全局 store 入口，维护：
  - `config`
  - `documentSessionSnapshot`
  - `recentList`
  - `externalFileChange`
  - 兼容字段 `fileName`、`saved`

### Renderer 文档会话链路

- 文档真相由 `wj-markdown-editor-web/src/util/document-session/documentSessionSnapshotUtil.js` 统一归一化。
- 编辑页 `wj-markdown-editor-web/src/views/EditorView.vue` 通过以下模块驱动：
  - `rendererDocumentCommandUtil.js` 发送 `document.edit`、`document.save`、`document.save-copy`、`document.get-session-snapshot`
  - `rendererSessionSnapshotController.js` 协调首次加载、激活重放、缺失最近文件提示
  - `rendererSessionActivationStrategy.js` 处理 keep-alive 激活恢复策略
  - `createRendererSessionEventSubscription()` 监听 `document.snapshot.changed`
- 当前编辑流不要再新增绕过 `document-session` 的旧式状态通道。

### Electron 文档运行时链路

- 应用入口 `wj-markdown-editor-electron/src/main.js` 负责：
  - 初始化 `documentSessionRuntime`
  - 注册 `wj://` 自定义协议
  - 配置单实例锁
  - 启动时打开文件、恢复最近文件或创建草稿窗口
- `wj-markdown-editor-electron/src/util/document-session/documentSessionRuntime.js` 是统一 runtime 入口：
  - `document.edit`
  - `document.save`
  - `document.save-copy`
  - `document.request-open-dialog`
  - `document.open-path`
  - `document.get-session-snapshot`
  - `document.external.apply`
  - `document.external.ignore`
  - `document.resource.open-in-folder`
  - `document.resource.delete-local`
  - `resource.get-info`
- `wj-markdown-editor-electron/src/util/document-session/windowLifecycleService.js` 负责窗口创建、关闭链路、外部文件监听、保存等待、最近文件维护与 BrowserWindow 生命周期。

### 资源操作主线

- 预览区资源菜单位于 `wj-markdown-editor-web/src/components/editor/PreviewAssetContextMenu.vue`
- Renderer 侧资源删除相关逻辑位于：
  - `previewAssetSessionController.js`
  - `previewAssetDeleteDecisionUtil.js`
  - `previewAssetDeleteConfirmController.js`
  - `previewAssetRemovalUtil.js`
- 主进程资源处理统一经 runtime 与资源服务完成，不要在 Web 端自行拼接平台路径或直接假设删除权限。

### 外部文件变更主线

- 配置项 `externalFileChangeStrategy` 位于默认配置 `wj-markdown-editor-electron/src/data/defaultConfig.js`
- 取值：
  - `apply`：自动应用磁盘最新内容
  - `prompt`：弹出差异对比，由用户选择忽略或应用
- 差异对比弹窗组件为 `wj-markdown-editor-web/src/components/ExternalFileChangeModal.vue`，基于 `diff` + `diff2html` 生成并排对比视图。

## 技术栈

### Web 前端（`wj-markdown-editor-web`）

| 类别 | 技术 |
|------|------|
| 框架 | Vue 3.5（Composition API + `<script setup>`） |
| 构建工具 | Vite 6 |
| 路由 | Vue Router 4 |
| 状态管理 | Pinia 3 |
| UI 组件库 | Ant Design Vue 4 |
| 样式方案 | UnoCSS + SCSS |
| 编辑器 | CodeMirror 6 |
| Markdown 解析 | markdown-it 14 + 自定义扩展 |
| 图表渲染 | Mermaid 11 |
| 差异对比 | diff + diff2html |
| 公式渲染 | KaTeX |
| 国际化 | vue-i18n 11 |
| 测试 | Node 内置 `node:test` |

### Electron 桌面端（`wj-markdown-editor-electron`）

| 类别 | 技术 |
|------|------|
| 框架 | Electron 39.2.7 |
| 构建工具 | electron-builder 26 |
| 自动更新 | electron-updater |
| 日志 | electron-log |
| 截图 | electron-screenshots |
| 文件操作 | fs-extra |
| 调度 | node-schedule |
| 测试 | Vitest 4 |

## Markdown 能力


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nlbwqmz/wj-markdown-editor](https://github.com/nlbwqmz/wj-markdown-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
