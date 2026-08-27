---
trigger: always_on
description: 本文件给任何在此仓库工作的 agent（DSH / 莲 / Pi 等）在动代码前必读。
---

# AGENTS.md — dsh-persona-memory 开发约定

本文件给任何在此仓库工作的 agent（DSH / 莲 / Pi 等）在动代码前必读。

## ⚠️ 首要规范：宿主包必须 peerDependencies（违反会炸掉全部工具）

- **`@deepseek-ai/*` 宿主包**（`dsh-tools`、`dsh-llm`、`dsh-home-paths` 等）**只允许出现在 `peerDependencies`**，**绝不写进 `dependencies`**。
- **为什么**：`dsh plugin add`（=pnpm）会把 `dependencies` 真装进 `<profile>/node_modules`，与宿主 dsh 安装目录形成**双物理副本**；而 `dsh-tools` 的 `TOOL_RUNTIME_SCHEDULER` 是普通 `Symbol()`（非 `Symbol.for()`，见 `dsh-tools/lib/index.js:2409`），双副本互不认 → 执行循环读不到调度器 → **所有工具调用崩** `Cannot read properties of undefined (reading 'prepare')`（web profile 必现）。
- **为什么 peer 就够了**：兜底目录 `$DSH_HOME/profiles/node_modules`（每次启动自动维护的 junction 集合）保证 peer 声明下解析自动回落宿主副本——不需要 junction、不需要加载期自检，**peer 是唯一必需的声明**。
- **安装后验证**（在 `<profile>` 目录下跑）：

  ```bash
  node --input-type=module -e "
  import { createRequire } from 'node:module';
  const require = createRequire(import.meta.url);
  console.log(require.resolve('@deepseek-ai/dsh-tools'));"   # 期望输出宿主路径（含 @deepseek-ai\dsh\node_modules\@deepseek-ai\dsh-tools）
  ```

- 若安装后 `<profile>/node_modules/@deepseek-ai/` 出现真副本（诱因：`autoInstallPeers: true` 或误写进 dependencies）→ 直接删除副本即可，兜底会接管解析。

## 项目事实

- **用途**：DSH 的持久化长期人格记忆插件——`memory`（read/add/update/delete/rewrite）+ `memory_search` 工具，每请求把记忆注入 system prompt（order 55），带内容扫描（秘密/提示注入拦截）。
- **自动能力**（`lib/learning.ts` + `lib/consolidate.ts` + `lib/llm-helper.ts` + `lib/correction.ts`）：后台学习（每 `learnIntervalTurns`=10 轮用会话自身 LLM 路由复习近期对话并 `store.add` 事实）+ 纠正检测（`feedback/record` + **对话内模式检测**——强/弱/负向模式 + 指令词，仅 `source.kind==='user'` 直接人工消息，`correctionRateLimitTurns`=3 限频）→ 存 `[correction]` 失败条目 + 用量提示（`usageNudgeThreshold`=0.9）+ **自动合并**（超限触发 LLM 合并，严格更小/可解析/过扫描才经 `replaceEntries` 提交，失败不损坏记忆）。
- **FTS5 记忆镜像**（`lib/fts.ts`）：`<dir>/.memory-index.sqlite`，动态 import node:sqlite（不可用则 `memory_search` 回退子串扫描），按文件 mtime 自动重建，literal-phrase 查询（FTS5 语法当数据处理）。
- **向量记忆镜像**（`lib/vector-index.ts` + `lib/embedding.ts`）：`vectorIndexDir`（默认 `$DSH_HOME/memory`）下 `.memory-vec.sqlite`，**只存 embedding 数据**（entries 表：which/text_hash/text/embedding BLOB/created/last，UNIQUE(which,text_hash)；meta 表 schema_version）；**指纹增量同步**（sha256 每条原始 § 条目，只 embedding 变化项，Pi 改共享文件自动跟上）；检索纯 JS 余弦（条目少，不装 sqlite-vec）；provider 二选一：`remote`（OpenAI 兼容 `/embeddings`，`embeddingBaseUrl`+`embeddingApiKey` 或 `DSH_EMBEDDING_API_KEY` env；DeepSeek 官方 embeddings 接口状态不明勿依赖）或 `local`（**`@xenova/transformers` 已进 dependencies**，首次使用自动从 HuggingFace 下载模型到 `embeddingCacheDir`（默认 `$DSH_HOME/models`），`embeddingRemoteHost` 可换镜像如 `https://hf-mirror.com`，下载进度经 ctx.logger 记录）；`vectorEnabled` 默认关，开后在 `memory_search` 走 **hybrid（FTS5 + 向量 RRF 融合）**，无向量则 FTS5/子串降级；注入成本与 FTS5 同级（仍 ≤searchMaxResults、只进工具输出）。
- **常驻指令**（`lib/standing.ts` + `lib/standing-command.ts`）：STANDING.md 每行一条、硬预算（20 条/2000 字符）、始终注入（order 50，早于记忆块 55）；只有 `/standing` 用户命令或直接编辑能写（模型无写入口，防注入）。
- **失败记忆**（`lib/failures.ts`）：failures.md（hermes 文件名）§ 格式、结构化 `[category] 内容 — Failed: … — Corrected to: …`、最近 7 天/5 条注入（order 52）、纠正检测落 failure 并去重；`which: 'failure'` 字符上限=memory×2。
- **项目级记忆**（`lib/projects.ts`）：projects-memory/<项目名>/MEMORY.md（Pi 兼容根：dir 为 hermes 数据目录时解析到 ~/.pi/agent/projects-memory）；git 仓库根为项目身份（worktree 共享）+ 迁移桥；按会话 cwd 自动注入（order 53）；`project` 参数需过 `safeProjectName` 防路径穿越；项目合并用 projectCharLimit。
- **与 Pi 共享记忆**：`MEMORY.md` / `USER.md` 磁盘格式与 pi-hermes-memory 完全兼容（`§` 分隔 + 行尾 `<!-- created=, last= -->` 注释）；`dir` 默认自动指向 `~/.pi/agent/pi-hermes-memory`（存在时），否则 `$DSH_HOME/memory`。
- **WebUI 记忆管理页**（`lib/admin.ts` + `client/src/main.ts`，0.2.0）：Host 用 `webServer` 注册 `/api/persona-memory/*` 路由（status/checkModel/configSave/update/delete/standingAdd/standingUpdate/standingRemove/rebuildVector/projectUpdate/projectDelete/backup/restoreLatest，**loopback-only 防护**，同 dsh-ssh 围栏）；Client 源码在 `client/src/main.ts`（TS，纯 React.createElement），由 `scripts/build-client.mjs`（esbuild，external react）构建成**预构建 bundle** `client/client.js`（`window.__ModuleLoader__.load` 格式，经 `dsh.client` + `exports["./client"]` 声明加载，**非动态插件无 harness.handle**，用同源 fetch 调路由）；页面注册到 `settings.section` slot（id=persona-memory，order=25，label=记忆管理）；卡片=常驻指令/记忆文件/项目记忆/检索索引/本地备份/插件配置；**检索索引卡=纯状态**（FTS5/向量大小、重建按钮，**不含任何配置项**——向量配置统一在插件配置卡，与 v27 定稿一致）；**插件配置卡**按 `CFG_SCHEMA`（40 项，分组：基础/学习/合并/检索/向量/项目/常驻/失败/管理/备份）schema 驱动渲染（bool→checkbox、number→数字框、select→按钮组、string→输入框），**embeddingModel 特殊渲染**：输入框+缓存检测+已下载模型按钮组（点击选用），保存写回当前 profile 的 `cordis.patch.yml` 中 persona-memory 段（`adminProfile` 配置项指定 profile 名，默认 web）；**本地备份卡**：备份目录 `$DSH_HOME/memory-backup/latest/`（与 Pi 完全隔离、只保留最新快照，8 文件=MEMORY/USER/failures/STANDING + projects-*.md），手动「立即备份/从最新备份恢复」；**自动保护**（`lib/admin.ts` `startAutoProtect`，index.ts 启动，ctx.timer.interval 每分钟 tick）：按 `autoBackupMin`（默认 60，0=关）间隔覆盖备份 + Pi 丢失检测（`autoSwitchOnPiLoss`，Pi 曾见 MEMORY.md 后消失 → patch 写 `dir: $DSH_HOME/memory` + restoreFromLatest）；**Client 源码修改后必须 `npm run build:client` 重新构建 + 重装 + 重启 web 才生效**（预构建产物按包分发）；client 只依赖 react + 浏览器 fetch，**不 import 任何 @deepseek-ai client 包运行时值**（peerDependencies 已声明 react、dsh-client-runtime、dsh-client-ui-slots、dsh-host-webserver 等）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Quophic/dsh-persona-memory](https://github.com/Quophic/dsh-persona-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
