---
trigger: always_on
description: oh-my-dsh 是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（下称 DSH）的桌面化 monorepo：出树插件与 Electron 壳同仓、独立发版。两个平面：
---

# AGENTS.md

oh-my-dsh 是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（下称 DSH）的桌面化 monorepo：出树插件与 Electron 壳同仓、独立发版。两个平面：

- **`plugin/<name>/`** —— 可独立安装、独立打 tag 的 DSH 插件包。成员：`dsh-desktop-bridge`（桌面门控：外链路由、原生注意力通知、桌面指示）、`dsh-mcp-settings`（2026-08-19 subtree 迁入）、`dsh-provider-balance`（2026-08-19 subtree 迁入，纯 DOM 注入）、`dsh-reasoning-efforts`（2026-08-20 新写，host-only：给手写 llm-pi-ai 模型补 `reasoningEfforts` 声明；0.2.0 起规则可附带 `compat` 开关（`supportsReasoningEffort`/`thinkingFormat`，zai 系路由自动探测为关、必须显式声明档位才上线），efforts 与 compat 两片按「缺什么补什么」独立判定、目录继承只豁免 efforts 片，契约见包内 README，决策见 `docs/notes/2026-08-20-reasoning-efforts.md` 与 `docs/notes/2026-08-27-reasoning-efforts-compat-fill.md`）、`dsh-web-search-toggle`（2026-08-20 新写，双面：通用设置页「Web Search」开关——DEEPSEEK_API_KEY 状态提示 + home patch 持久化 + Host assembly/guard 跨 Agent Preset 关闭原生 `web_search`，契约见 `docs/notes/2026-08-20-web-search-toggle.md` 与 `docs/notes/2026-08-21-web-search-toggle-preset-scope.md`）、`dsh-compaction-hierarchical`（2026-08-20 新写，host-only：继承 stock compaction 事务，以有界 map-reduce 让小上下文模型压缩大历史；0.1.3 起作为官方 upstream/既有用户 preset 的兼容 Provider 保留，Oh My DSH 默认由 fork stock basic 接管；契约见包内 README，决策见 `docs/notes/2026-08-20-hierarchical-compaction.md` 与 `docs/notes/2026-08-22-stock-hierarchical-compaction.md`）、`dsh-branding`（2026-08-21 新写，browser-only：占用 `sidebar.brand.name` 替换字标为 "Oh My DSH"+"Harness" pill 并重写 document title——**始终挂载、无桌面门控**，终端/浏览器/桌面同一条路；决策见 `docs/notes/2026-08-21-branding-plugin.md`）、`dsh-fs-observation-log`（2026-08-21 新写，host-only：持久化 `fs/observed` evidence 到 `$DSH_HOME/fs-observation-log/`，进程重启/fork 后在 edit/write 前置阶段按「版本 token 仍相等」恢复观察记录，消除 `FS_NOT_OBSERVED` 误伤且不削弱 stock guard——preset 行挂载、patch 刻意为空、零 harness 运行时 import；契约见包内 README，决策见 `docs/notes/2026-08-21-fs-observation-log.md`）、`dsh-model-image-input`（2026-08-21 新写，browser-only：以 dsh-provider-balance 同类纯 DOM 注入姿势，在 stock「设置→模型→供应商→自定义设置」的每条已保存 pi-ai 模型行内插入图片输入按钮；非原生三态菜单（跟随默认/仅文本/文本+图片）固定 196px、向左对齐并按实际高度上下翻转，经 `settingsScope` + revision-fenced `settings.mutate` 写整组 `models` 数组、即时生效；中英文 aria/action 双锚排除 DeepSeek 卡片，DOM 失配 fail-invisible，client 半零 `@deepseek-ai/*` 值导入；契约见包内 README，决策见 `docs/notes/2026-08-21-dsh-model-image-input.md`）、`dsh-send-while-running`（2026-08-22 新写，browser-only：占用已声明加性槽 `conversation.input.right`（list、replaceRisk none），在普通会话 running 且草稿有内容时于 stock Stop 左侧渲染一个 Send 孪生按钮——点击走 session standard kit 的 `inputActions.submit()`（与 stock Send 同一条 queue 投递公共路径），可见性逐项镜像 stock `primaryStops`/`empty` 定义、排除 continuable 子会话与 removed；按钮 `order:1` + `:has()` 作用域把 stock 主按钮 `order:2`，卸载即回到 stock 布局，锚点只用 `[data-slot]` 接缝与 `button:last-of-type`（不引 stock CSS-module 类名）；ui-conversation/locale 仅 type-only import（client bundle 唯一 require 是 react/jsx-runtime，无需 runtime peer 链接），无桌面门控，终端/浏览器/桌面同一条路；契约见包内 README，决策见 `docs/notes/2026-08-22-dsh-send-while-running.md`）、`dsh-model-efforts-editor`（2026-08-27 新写，browser-only：dsh-model-image-input 同类纯 DOM 注入姿势，在 stock 模型设置卡每条 pi-ai 模型行内注入推理档位按钮与内联编辑弹层——三态（跟随默认 / 不推理 / 自定义档位+线上值）+ Z.ai compat 勾选（`supportsReasoningEffort`/`thinkingFormat: zai`），经 `settingsScope` + revision-fenced `settings.mutate` 写整组 models 数组即时生效；上游 stock 刻意不提供 provider 级 effort 控件而 composer 选择器又依赖已有声明，本插件补上 per-model 编辑入口；锚定规则与 fail-invisible 边界同 model-image-input；契约见包内 README，决策见 `docs/notes/2026-08-27-dsh-model-efforts-editor.md`）
- **Electron 壳** —— spawn harness sidecar、端口分配、就绪检测、窗口加载。壳层不含业务逻辑；harness 不感知壳的存在。业务集成只特殊对待桥插件（gate + IPC）；分发层另按本文件的 desktop-owned 清单打包/安装层次压缩与 Web Search 开关，但不读取其 Provider/策略业务。0.2.x Tauri 壳已删除，只留 `scripts/tauri-cutover-latest-json.mjs` 给仍在轮询旧端点的客户端。

规范层级：[README.md](README.md) 记录「为什么」（技术选型）；本文件记录「契约与约定」（怎么做）；代码是实现。冲突时以本文件为准。改契约必须同 PR 改本文件。

## Repository layout

```
plugin/<name>/               一个可独立发版的 DSH 插件包（目录名 === package.json name）
  dsh-desktop-bridge/        桌面门控桥 + 日志汇（本文件「插件契约」一节）
    src/index.ts             host half：surface 插件，空 apply
    src/log-sink.ts          日志汇 host 行：ctx.logger → 每启动一个 JSONL 文件（见「日志汇行」）
    src/invariant.ts         伙伴不变量说明
    src/client/              browser half：env 探测 + 三个桥 + shell.overlay 桌面指示 + 标题栏融合
    tests/                   node:test 单测（纯函数）
src/                         Electron 壳（main / preload / sidecar / profile CAS；图标与 DMG 背景）
scripts/                     壳层与工具脚本：prepare-runtime.mjs、prepare-desktop-bundle.mjs、build-electron.mjs
docs/                        packaging-playbook.md + notes/（决策记录住仓根，不跟包走）
```

`CLAUDE.md` 是指向本文件的 symlink（与 DSH 仓库同惯例）：改 AGENTS.md，不要改链接。

## 插件 monorepo 规范

本仓是「个人 DSH 扩展 + 桌面壳」的单一事实源。插件与桌面同仓，是为了一次 checkout、一次 harness rc bump 过全树，同时保留各自的发布节奏。对照：[dataelement/dsh-desktop](https://github.com/dataelement/dsh-desktop) 把 DSH 钉在 npm 上、用仓根 `patches/` 改上游压缩产物——那是壳侧补丁模型，不是插件布局，不学。

### 落点


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aka-danielZhang/oh-my-dsh](https://github.com/aka-danielZhang/oh-my-dsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
