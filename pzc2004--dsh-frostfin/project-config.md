---
trigger: always_on
description: > 写给 AI 编码代理的项目说明。读者对本项目一无所知。本文所有结论均来自对仓库实际内容的核实（2026-08）。
---

# AGENTS.md — dsh-frostfin（月芒霜鳍鲸）

> 写给 AI 编码代理的项目说明。读者对本项目一无所知。本文所有结论均来自对仓库实际内容的核实（2026-08）。

## 项目概述

**dsh-frostfin** 是一个 DeepSeek Harness（DSH）的 **agent loop 插件**：把 DSH 会话的驱动者整个换成 **Kimi Code 本人**——通过 ACP（Agent Client Protocol，JSON-RPC over stdio）直连 `kimi acp` 子进程，中间没有第二个 agent 转述。

```
用户 ──► DSH Web UI（主题 / 生态 / 会话管理 / 审批）
            │  session events
      frostfin（DSH 插件，占 agent loop 插槽）
            │  ACP（JSON-RPC over stdio）
      kimi acp 子进程 ── 全部思考、工具调用、回话
```

DSH 的架构里 agent loop 本身就是可替换插件（`ctx.agents.setFactory`，全局唯一）。frostfin 注册自己的工厂，把每个 DSH 会话桥接到一个 `kimi acp` 子进程，并把 ACP 事件流实时转译成 DSH 会话事件——流式回话、历史回放、审批弹窗都是 DSH 原生体验。

与 kimi-tide 等方案的区别：那些项目把 Kimi 接在模型层/工具层（DSH 主 loop 不变）；frostfin 把 loop 本身换成 Kimi Code。

**版本基线**：对锁 `@deepseek-ai/dsh@0.1.0-rc.6` 与 Kimi Code 0.36.x，两边都在快速迭代。包管理器为 pnpm，Node.js 要求 ≥ 22.19。

## 技术栈

- **语言**：TypeScript（strict 模式，ESM，`"type": "module"`，`module: NodeNext`），编译目标 ES2022。
- **插件框架**：`@deepseek-ai/cordis` 4.0.1（peer 依赖）——一切注册走 Cordis effect，卸载即撤销。
- **ACP 客户端**：`@agentclientprotocol/sdk`（`ClientSideConnection` + ndjson stdio）。
- **配置模式**：`@deepseek-ai/schemastery`（`src/index.ts` 的 `Config`；package.json 还声明了 zod，但 src 当前未引用）。
- **浏览器半身**：React（TSX，自动 JSX 运行时），react 等平台模块由 DSH 外壳注入（构建时 external）。
- **构建**：`tsc`（宿主半身 src → lib）+ `esbuild`（浏览器半身 src/client → lib/client.js）。
- **测试**：Node 内置 `node:test`，纯 `.mjs` 文件直接驱动构建产物 `lib/`。
- **浏览器实测**：`playwright-core`（仅 `scripts/` 下的手工探针用）。

## 构建与测试命令

```sh
pnpm install        # 安装依赖（pnpm-lock.yaml 锁定）
pnpm build          # = build:host + build:client（完整构建）
pnpm build:host     # 仅 tsc：src/ → lib/（排除 src/client/）
pnpm build:client   # 仅 esbuild：src/client/index.ts → lib/client.js（CJS bundle，包进 window.__ModuleLoader__.load 协议）
pnpm test           # node --test test/*.test.mjs
```

**重要**：测试直接 import `../lib/*.js`（构建产物），改完 `src/` 必须先 `pnpm build` 再 `pnpm test`，否则测的是旧代码。

最近一次验证：`pnpm build` 通过，`pnpm test` 96 个测试全部通过（约 20 秒，含真实子进程的集成测试）。

## 目录结构与模块划分

```
src/                  宿主半身（Node 侧，tsc 编译到 lib/）
  index.ts            插件入口：name/inject/Config/apply；inject = ['agents','sessions','subprocess']
  factory.ts          FrostfinAgentFactory：AgentFactory 实现，占 loop 插槽；按 preset 分发（frostfin → kimi，其他 → 影子原生 loop）
  agent.ts            FrostfinAgent：一个 DSH 会话 ↔ 一个 kimi acp 子进程 ↔ 一个 ACP 会话；turn/step 驱动纪律
  acp-process.ts      kimi acp 子进程管理 + ClientSideConnection 封装（spawn → 握手 → prompt → dispose 阶梯）
  translate.ts        ACP session/update → DSH 会话事件的纯转译层（零 I/O，可单测）
  permission.ts       M2 审批桥：ACP request_permission → DSH ctx.approval（fail-closed）
  question.ts         M7 提问通道：kimi AskUserQuestion 复用审批通道，插件自建模态框闭环
  commands.ts         斜杠命令：/frostfin-* 系列 + /yolo /auto 快捷键 + kimi 内建透传（/compact /status /usage /mcp /tasks /help）
  kimi-route.ts       名义模型路由 provider 'kimi-code'（喂饱 DSH 模型层两道门禁）+ KimiModelCatalog（真实模型目录，持久化）
  kimi-sessions.ts    DSH↔kimi 会话绑定映射（KimiSessionMap）与运行档位记忆（KimiSessionPrefs），JSON 文件存 ~/.frostfin/
  config-sync.ts      把 DSH 已配置的模型供应商同步进 kimi config.toml 的托管标记块
  preset-install.ts   把「月芒霜鳍鲸」preset 复制到 $DSH_HOME/.agent-presets/frostfin
  shadow-native.ts    影子挂载原生 agent-loop（cordis isolate + Proxy 捕获其工厂，供 preset 分发委托）
  host-scope.ts       从宿主进程模块树解析 dsh-* 包（模块私有 Symbol 必须与宿主同一份）
  panel.ts            webServer HTTP 端点：/plugins/frostfin/*（会话列表/打开/状态条/提问/远程主机）
  remote.ts           远程线：ssh+tmux shim 命令构建、远程体检、活 TUI 探针（双写防护提示）+ HostDriver 宿主驱动接口（hostDriverFor 分派点：本地/远程一视同仁，posix-local 与 posix-ssh-tmux 双实现，Windows 将来在此分派）
  ssh-config.ts       ~/.ssh/config 解析（OpenSSH/VS Code 语义：Host 块、Include、first-obtained-wins）
src/client/           浏览器半身（React TSX，esbuild 打包；tsc 排除此目录）
  index.ts            槽位注册：会话面板 tab、「文件」文件树 tab、状态条 dock、提问模态框、输入区工具行按钮（thinking/权限模式/传文件/折叠步骤）、@ 工作区文件补全 source（inputTriggers 流水线）
  SessionsPanel.tsx   「月芒霜鳍鲸」tab：本地/远程 kimi 会话列表与接入
  FilePanel.tsx       「文件」tab：会话工作区文件树（懒加载；点文件复制相对路径；本地/远程一视同仁）
  FoldStepsPill.tsx   「折叠步骤」开关：CSS 钩子（data-tool / data-variant="think"）隐藏 Think 与工具行；DSH 改钩子名则静默失效（无害降级）
  collapse-nodes.ts   单条消息折叠：长输入/输出折成一小段（整壳限高 10em + 渐变 + 壳内绝对定位按钮——slot 包装是 display:contents，壳自身才是可压的盒）；折叠后抵消 DSH 底部吸附回拨并把被折消息定位到屏幕中央（立即一次 + 200ms 再咬一次）；MutationObserver 自愈
  StatusDock.tsx      输入框下方状态条（模型/thinking/模式/git 分支/上下文占用/Kimi Coding 配额/cwd，3 秒轮询）
  UploadPill.tsx      输入区「传文件」按钮：本机文件 scp 到远程会话的服务器（仅远程会话显示），带实时进度条
  QuestionModal.tsx   AskUserQuestion 多选模态框
presets/frostfin/     「月芒霜鳍鲸」preset 定义（preset.yml + agent.cordis.yml，最小 persona 行）
cordis.patch.yml      DSH profile 补丁：insert frostfin 行、agent-presets 默认改为 frostfin、禁用 agent-loop
test/                 node:test 测试（.mjs）+ fixtures/scripted-acp-child.mjs（script 化 ACP 子进程夹具）
scripts/              build-client.mjs 是真构建步骤；probe-*.mjs / spike-*.mjs / ui-*.mjs 是手工探针
docs/                 design-v0.1.md（设计稿）、guide.md（用户手册）、upstream-kimi-acp.md（上游需求清单）
reference/            本地参考仓库（deepseek-harness、kimi-code、kimi-tide），gitignore 排除，仅供对照阅读
lib/                  构建产物，gitignore 排除
assets/               图片素材（含《原神》版权素材，不在 MIT 覆盖范围）
```

## 运行时架构要点

- **换 loop = 换工厂**：`ctx.agents.setFactory(factory)` 全局唯一，`cordis.patch.yml` 同时禁用 `agent-loop` 行。所有入口（Web host、headless 等）都经 `ctx.agents.create()/resume()`，换工厂即全入口换驱动。
- **惰性启动**：新建会话不起 kimi 进程；首个 prompt 才 spawn + ACP 握手（initialize + newSession）+ 登记绑定。kimi 未登录/未装不挡开会话，问题在发送时以对话内指引浮现。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pzc2004/dsh-frostfin](https://github.com/pzc2004/dsh-frostfin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
