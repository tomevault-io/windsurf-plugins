---
trigger: always_on
description: > 本文件会被 DeepSeek Harness 的 `dsh-agent-instructions` 插件自动加载（`AGENTS.md` / `CLAUDE.md` 候选，从项目根到会话 cwd 逐级生效），作为在本仓库工作的 agent 的持久指引。它不覆盖 system / developer / 用户直接指令，只是帮助你更快定位答案。
---

# AGENTS.md — dsh-channels

> 本文件会被 DeepSeek Harness 的 `dsh-agent-instructions` 插件自动加载（`AGENTS.md` / `CLAUDE.md` 候选，从项目根到会话 cwd 逐级生效），作为在本仓库工作的 agent 的持久指引。它不覆盖 system / developer / 用户直接指令，只是帮助你更快定位答案。

## 1. 项目定位

这是 DeepSeek Harness 的即时通讯渠道插件（**社区项目，非官方**）：微信 / QQ / 钉钉 / 飞书 / Telegram，通过统一的 `ctx.channels` API 收发消息。仓库即 `dsh-channels`。

处理任何「本项目」的问题前，先读：

- `README.md` —— 安装、渠道配置与登录、渠道总览、开发命令、文档索引（最全入口）。
- `apps/example/minimal-profile/` —— 各渠道 patch 的完整示例；回答配置字段问题以它和 `README.md` 为准，**不要凭空编造字段**。

文档目录（按需加载，顶部 front-matter 标注了 `when_to_use` / `authoritative`）：

| 文档 | 何时读 |
| --- | --- |
| `docs/architecture.md` | 理解整体架构、判断依赖方向、核对架构红线（红线权威） |
| `docs/architecture/common-design.md` | 改 `channel-core` / `channel-harness` / `channel-control` / `channel-web` / `channel-files` 代码前（Contract/Bridge/控制面/Bundle 权威） |
| `docs/architecture/channel-roadmap.md` | 评估新渠道 / 扩展方向 / Channel-vs-Tool 边界 |
| `docs/adapter-authoring.md` | 新增第三方适配器、写 manifest、跑 `pnpm verify` |
| `docs/release.md` | 发版 / changeset / release gate（版本与发布权威） |
| `docs/weixin-live-verification-runbook.md` | 执行 Weixin live gate、填 manifest 真实值 |
| `docs/architecture/adr/` | 架构决策记录（上游边界、图片模型降级） |
| `.agents/skills/dsh-channels-verification/SKILL.md` | 核验渠道实现 / SDK / 官方接口 / 权限 / manifest / 上游版本漂移时使用 |

> **硬规则**：当任务涉及以下内容时，**必须**加载 `.agents/skills/dsh-channels-verification/SKILL.md`：
>
> - 核验 QQ / 微信 / 钉钉 / 飞书 / Telegram 实现
> - 修改渠道 SDK / API / 权限
> - 修改 manifest / testedVersion / upstream
> - 检查平台权限、事件订阅、Gateway intents
> - 判断 channel-web 权限展示是否与真实平台能力一致
>
> 这样即使某些 Agent 的 Skill 自动发现机制偶尔不工作，AGENTS.md 仍然会兜底要求它读取该 Skill。

### 入站访问控制

涉及以下内容时：

- inbound / `message.received`
- sender / conversation identity
- owner / owner claim
- dmPolicy / groupPolicy / allowFrom / requireMention
- interaction security

必须先阅读：

1. `docs/security/inbound-access-control.md`
2. `docs/security/channel-identity-map.md`
3. `docs/architecture.md`（涉及依赖/职责变化时）

Adapter 不得自行定义不同于 `channel-harness` 的 ACL 语义；外部主体必须先授权，再产生本地副作用（架构红线 13）。

包结构（`packages/`）：

| 包 | 职责 |
| --- | --- |
| `channels` | 对外 bundle（`@wsz987/dsh-channels`） |
| `channel-core` | Channel Contract / `defineChannelAdapter` |
| `channel-harness` | 渠道 ↔ Harness 桥 |
| `channel-files` | 可选通用文件扩展（存储 / 解析 / `read_channel_attachment`） |
| `channel-weixin` / `-qq` / `-dingtalk` / `-lark` / `-telegram` | 五个内置渠道适配器（Telegram：Bot API 长轮询 + edit streaming + getFile 下载） |
| `channel-compat` / `-testkit` / `-verify` / `-web` | 校验 / 测试 / 契约验证 / Web 可视化 |

常用命令：`pnpm build`、`pnpm typecheck`、`pnpm test`、`pnpm channels`、`pnpm channels:clean`、`pnpm verify <dir>`、`pnpm doctor`、`pnpm check:fixtures`、`pnpm check:manifests`、`pnpm check:upstream`（含 dsh-* 基线 `check:harness-compat`；`check:harness-newer` 为非阻塞提示，基线事实来源是 `scripts/check-upstream.mjs` 的 `HARNESS_TESTED_VERSION`）。

## 2. DeepSeek Harness 怎么排查（本体问题）

Harness 本体是**闭源但可读的发布产物**：源码随 npm 包以 `lib/` 分发（带完整 JSDoc），包内 `README.md` / `README.zh.md` 双语。绝大多数「它到底怎么工作」都能在本机读源码得到答案。

排查顺序：

1. **定版本**：`dsh --version`（报告问题必须带版本号）。
2. **读安装后的源码**：`node_modules/@deepseek-ai/*`。从包名猜实现位置，例如 `dsh-workspace`（工作区/归档）、`dsh-session-persistence-jsonl`（会话落盘）、`dsh-storage-json`（JSON 存储）、`dsh-host-apiproxy`（RPC API 契约）、`dsh-client-ui-*`（前端）、`dsh-web-app`（web bundle）。每个包的 `package.json` 有 `repository.directory` 指向 monorepo 目录，`README.md` 常引用仓库相对路径。
3. **排配置**：`dsh --help`（启动器 flag）、`dsh web --help`（web 应用 flag）、`dsh --profile web --dump-config`（合并后完整配置，含用户层与 `--patch`）、`--dump-default-config`（仅 bundle 层）。配置层优先级：各 bundle patch → profile `cordis.patch.yml` → `$DSH_HOME/cordis.patch.yml` → `--patch`；patch 是**整体替换**目标行 config，不是合并。
4. **查数据目录** `~/.dsh`（`$DSH_HOME` 可覆盖；Windows 下 `%USERPROFILE%\.dsh`）：

| 路径 | 内容 |
| --- | --- |
| `sessions/<encoded-cwd>/<id>/session.jsonl.zstd` | 会话日志（zstd） |
| `storages/workspace.json` | 工作区注册表 + **归档集合 `global.archivedSessionIds`** |
| `storages/session_projcache.json` | 会话投影缓存 |
| `settings.yaml` | 用户设置（热加载） |
| `.credentials.yaml` | 受管凭据（脱敏查看） |
| `profiles/<name>/` | 各 profile 的 `package.json`、`cordis.patch.yml` |
| `.anonymous-user-id` | 匿名用户 id |

常见定位：

- 会话在侧边栏「消失」 → `storages/workspace.json` 的 `archivedSessionIds`（当前 rc 版归档单向，删掉对应 id 并重启即恢复，日志未删）。
- 插件没生效 → `--dump-config` 里有没有对应行 id；bundle 是否进了 `dsh.profile.bundles`。
- 凭据/key 不生效 → `settings.yaml` 的 `llm-*:` 段 + `.credentials.yaml`。
- 启动即退出 → 完整报错 + `--dump-config`；launcher flag 必须在应用参数之前。

## 3. 官方文档与源码链接

DeepSeek Harness 官方文档站点（**优先查这里**）：

- 站点根：<https://deepseek-harness.github.io/deepseek-harness/>
- **Reference（CLI / 配置 / flag 参考）**：<https://deepseek-harness.github.io/deepseek-harness/reference/>
- **Develop → Basic（开发基础，含 AGENTS.md 编写约定等）**：<https://deepseek-harness.github.io/deepseek-harness/develop/basic/>

源码与仓库：

- 官方仓库：<https://github.com/deepseek-ai/deepseek-harness>
- CLI 包（npm）：<https://www.npmjs.com/package/@deepseek-ai/dsh>
- 本项目仓库 / issue：<https://github.com/wsz987/dsh-channels>

> 小技巧：任意 `@deepseek-ai/<pkg>/package.json` 的 `repository.url` + `repository.directory` + 包内 README 引用的相对路径（如 `reference/README.md`、`src/args.ts`）三者合起来，即可定位到官方源码/文档的确切位置。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wsz987/dsh-channels](https://github.com/wsz987/dsh-channels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
