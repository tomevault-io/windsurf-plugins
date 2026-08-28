---
trigger: always_on
description: 给要改这个仓库的人（包括编码代理）看的约定。读完再动手。用户向说明见 [README.md](./README.md)。
---

# AGENTS.md

给要改这个仓库的人（包括编码代理）看的约定。读完再动手。用户向说明见 [README.md](./README.md)。

## 这是什么

dsh-liketavern 是 DeepSeek Harness（dsh）插件，把 `dsh web` 做成 SillyTavern 式的角色扮演前端。角色卡（V1/V2/V3，PNG/JSON）、提示词预设、世界书、人设、正则、BM25 长期记忆、世界状态变化层，以及可回滚的楼层操作，都建在 dsh 的 agent 运行时上，而不是另起一套发信通道。

和 SillyTavern 最不一样的四件事，改代码时请按这个想：

1. **资产文件化**。导入的卡、世界书、预设落成工作区文件。agent 按需按条读，不要每轮全塞进 prompt，也不要把整本 JSON 灌进工具结果。
2. **多步思考**。一轮回复可以走多步 agent 循环：缺设定再检索，必要时写记忆或更新世界状态，最后一步才出正文。默认直接扮演，不要每轮例行调工具。
3. **楼层事务**。用户对某一层触发的写入（记忆、世界状态、定时器）必须能撤销。工作区写入走 WAL（楼层号 + 序号），回退就是逆序回放。例外：记忆超容量时的异步压缩在 idle 期 `runMaintenance` 执行，`floor=null` 不记 WAL，回退不会撤压缩——那是无损整理，不增删剧情事实。
4. **不复制 ST 的一次性输入**。提示词走 dsh 的 system-prompt 组装瀑布（稳定段 + runtime context）。不要在前端拼包直发，也绝不要用 `complete` 段盖掉工具前缀。

实测环境：dsh `0.1.1-rc.2`（`@deepseek-ai/*` 同版本），Node 24，Windows（CI 在 ubuntu 上跑 build/test）。依赖 rc.2 特有宿主行为的地方集中列在「宿主版本注记与升级」一节。

**查平台行为先看官方文档**：<https://deepseek-harness.github.io/deepseek-harness/>（上手：[guide/quickstart](https://deepseek-harness.github.io/deepseek-harness/guide/quickstart)；插件开发：[develop/basic](https://deepseek-harness.github.io/deepseek-harness/develop/basic/)，含打包安装、profile/bundle、patch 层顺序）。涉及宿主机制（slot、profile、patch、typert、system-prompt 瀑布等）的判断以官网文档和宿主源码为准，不要凭记忆猜。

**查宿主源码看 npm 包**：`@deepseek-ai/dsh` 及其依赖（`dsh-agent`、`dsh-session`、`dsh-system-prompt` 等）在 devDependencies 里锁定精确版本，`npm install` 后直接读 `node_modules/@deepseek-ai/dsh/` 和兄弟包里的 `lib/` 与 `.d.ts`。本地没装时看 npm registry 的 Code 页：<https://www.npmjs.com/package/@deepseek-ai/dsh?activeTab=code>，依赖包同站换包名。注意 `@deepseek-ai/dsh` 本体版本与 peer 包版本可能不同，先看 `node_modules/.../package.json` 确认实际版本再读代码。

运行期角色卡、记忆、会话绑定在 `$DSH_HOME/dsh-tavern/`，不在本仓库。不要为了方便调试把真实卡拷进 git。测试用手写工厂数据。

## 技术栈与三面运行

- TypeScript ESM（`"type": "module"`），`module: NodeNext`，strict + `noUncheckedIndexedAccess`。
- 宿主是 dsh（cordis 容器），分三面：
  - **host**（`src/index.ts`）：注册设置命名空间 `dsh-tavern`、初始化数据目录、安装 agent 预设、提供 `tavern` 服务（`TavernService`）、注册 typert remote、听 `session/event` 维护楼层 WAL 和每 turn/step 缓存。
  - **agent**（`src/agent.ts`）：由插件安装的 `tavern` 预设挂载（`presets/tavern/agent.cordis.yml`，启动时替换 `__AGENT_MODULE__`）。在 `system-prompt/assemble` 写入稳定段 `tavern:standing` 和 runtime context `tavern:turn`；`agent/pre-step` 记 step；`agent/request` 合入采样，并把 thinking 映射成 `reasoningEffort`（模型元数据经 `resolveModelInfoCached` 进程内缓存）；注册 7 个模型工具；`agent/status` 转入 idle 时 `runMaintenance` 做记忆压缩。
  - **client**（`src/client/`）：浏览器 React UI。五块 slot：设置 `settings.section`、助手操作条 `conversation.chat.assistant-actions`、会话头芯片 `conversation.session.header.actions`、新会话英雄区 `conversation.input.dock`、助手排版 `conversation.chat.node`（`assistant-step`，priority -1）。经 `ctx.remote.$mount(TYPERT_REMOTE)` 挂 remote，调用时用 `ctx.get('remote.tavern')`。rc.2 起 chat.node 的 owner props 不再给 `loadImage`，图片走 `renderMessageImages({ images, align })`；`fileMentions` 是 owner 函数，要像原生 AssistantNodeView 那样用 turn-tail owner 解析后再传给 MarkdownText。
- host 和 client 用 typert RPC，契约在 `src/remote.ts`（`METHODS` 表 → 描述符）。方法返回裸业务值，失败抛错。`{ ok, value | error }` 信封由 gateway 生成，service 层不要再包一层。加删改一个方法要同步三处：`src/remote.ts` 的 `METHODS`、`src/node/service.ts` 的实现、`src/client/types.ts` 的 `TavernRemote`——最后这个是手工维护的契约镜像，漏改就编译不过。
- 运行时依赖只有 `zod`。直接使用的 `@deepseek-ai/*` 以 peerDependency 声明，由 dsh 宿主提供；开发依赖使用公开 npm 的精确版本。禁止 `file:`、本机绝对路径、junction 或符号链接依赖。host/client bundle 里平台模块一律 external。
- 设置用 schemastery（`src/node/config.ts`），命名空间 `dsh-tavern`，用户覆盖在 `~/.dsh/settings.yaml`，`applies: 'live'`。包括采样（含 thinking 档位）、世界书全局、记忆、新会话默认绑定 `defaults`、`interactiveCards`、`cardNetworkWhitelist`、`cascadeDeleteEmbeddedBook`（删卡时是否连同内嵌世界书，默认开；关掉则删卡前把内嵌书抢救到世界书库）。rc.2 起 web 设置 RPC 的命名空间白名单已移除，宿主通用设置文档页也能看到/改 `dsh-tavern` 的键——这是宿主行为，插件面板仍是主入口，不要为此改面板。

## 提示词通道与 agent 循环

live 路径不把整包 ST 预设塞进 system。`assemblePrompt` 按 Prompt Manager 算出全量序列（给预览），再拆成两条 dsh 通道：

| 通道 | dsh 落点 | 内容 | 稳定性 |
| --- | --- | --- | --- |
| standing | system 段 `tavern:standing`（order 210，工具说明 100–199 之后） | `BOUND_DISCIPLINE` + 角色定义 + 预设骨架 + 常驻世界书 + 静态深度注入（无本轮宏的 in-chat 条目 / depth_prompt） | 绑定不变则按会话 × 生成场景钉死字节（`STANDING_PIN_VERSION` + generationType + 卡/预设/人设指纹 + 资产修订号）。纪律或段布局变了必须递增版本，否则进程内旧钉死会挡住新文案。编辑/删除预设与世界书经 `TavernState` 写方法 bump 修订号（`standingRevTags`）。绕开 TavernState 手改文件不会被捕获。 |
| turn | runtime context `tavern:turn` | 固定 `TURN_PLAYBOOK`（不随 step 变）+ 关键词世界书/记忆/变化层/AN/本轮宏 | dsh 追加成 user 快照（`Current runtime context.`），盖住更早的同名快照；宿主对快照按字节去重——同轮后续步骤快照不变则不再追加（多步零快照开销）。步骤收口压力走 `【Tavern 步骤】` inject（见下）。 |
| messages | 仅「预览提示词」 | 完整 ST 序列（含 @D 真实插历史位置） | live 请求插不进会话日志中间；排查以预览为准。 |

时钟在 standing 里冻结。残留 `{{…}}` 写入 dsh 段前要 `neutralizeDshMustache`。未绑卡时不要删掉 `tavern:standing` 段，只换成 `UNBOUND_STANDING` 短文案，避免段布局抖动打穿 KV。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Amakurai/dsh-liketavern](https://github.com/Amakurai/dsh-liketavern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
