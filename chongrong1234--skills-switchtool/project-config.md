---
trigger: always_on
description: > 面向 AI 编码助手的项目指南。描述的是**当前代码的实际状态**;`PLAN.md` 是早期计划书(其中 Ink TUI、zod、Tauri 等均未落地,以本文件与 `README.md` 为准)。
---

# AGENTS.md

> 面向 AI 编码助手的项目指南。描述的是**当前代码的实际状态**;`PLAN.md` 是早期计划书(其中 Ink TUI、zod、Tauri 等均未落地,以本文件与 `README.md` 为准)。

## 项目概述

**Skills SwitchTool**(`skills-switchtool`,v1.8.0):项目中心化的 Agent Skills 管理工具。交互模式仿照 cc-switch:**中央存储 + 切换 + 写入目标工具配置位置 + 快照可回滚**。

核心概念:

- **中央库是唯一事实来源**:全部 skills 实体存放在 `~/.skills-switch/library/`(可用 `SSW_HOME` 环境变量覆盖,测试隔离用);MCP server 是纯配置,集中在 `mcps.json` 注册表(name 即唯一键)。
- **项目是一等公民**:项目档案(`projects.json`)记录 `项目 ↔ 技能集 ↔ MCP 服务集 ↔ 目标 agents` 绑定与 `activeProjectId`。
- **apply = 物化**:把项目技能集写入各 agent 的项目级 skills 目录(claude-code→`.claude/skills`、kimi-code→`.kimi-code/skills`、cursor→`.cursor/skills`、codex→`.codex/skills`、gemini-cli→`.gemini/skills`、windsurf→`.windsurf/skills`、roo-code→`.roo/skills`、qwen-code→`.qwen/skills`、trae→`.trae/skills`、factory-droid→`.factory/skills`、deepseek-harness→`.dsh/skills`、cline→`.cline/skills`、continue→`.continue/skills`、crush→`.crush/skills`;agents/copilot/opencode/openclaw/amp 五家项目级都指向通用 `.agents/skills`——Agent Skills 开放规范的互操作路径,同时启用时幂等跳过)。默认 symlink(库改动即时生效),可选 copy;symlink 失败自动降级 copy 并告警。同名冲突的既有内容先移入快照再覆盖。MCP 服务集**合并**写入各 agent 的项目级配置(claude-code→`.mcp.json`,kimi-code→`.kimi-code/mcp.json`,cursor→`.cursor/mcp.json`,codex→`.codex/config.toml` 的 `[mcp_servers.*]` 段,qwen-code→`.qwen/settings.json` 的 mcpServers 字段,trae→`.trae/mcp.json`,factory-droid→`.factory/mcp.json`):保留用户已有条目、同名覆盖,已存在的配置文件先整体进快照再写。
- **全局(用户级)共享应用**(`src/core/global.ts`):把选定 skills 物化到各 agent 的用户级 skills 目录(`~/.claude/skills` 等,即适配器的 `userSkillsDir()`),一次配置、该 agent 的所有项目共享。档案存 `global.json`,快照挂在固定名 `__global__` 下,复用 apply.ts 的物化/移除原语与 snapshot.rollback。**MCP 是项目级概念,全局共享只管 skills**。CLI(`ssw global`)、REST(`/api/global*`)、桌面 GUI(「全局共享」视图)、TUI(g 键)均已接入。
- **配置库导出/导入**(`src/core/profile.ts`):`ssw-profile@1` 单文件 bundle(skills 注册表 + MCP + 项目档案 + 全局档案 + local 技能实体 base64),跨机器/跨平台整体搬家;导入幂等——github 按仓库去重重克隆、local 文件落库带路径穿越防护、项目 id 冲突换新 id、`activeProjectId` 与全局档案仅在本机空缺时采用。
- **收养既有 skills**(library.ts `adoptFromAgent` / `adoptFromAllAgents`):把 agent 用户级/项目级 skills 目录里已存在的 skills 收进中央库(跳过指向库内的 symlink 与同名条目),先纳管再统一分发;`adoptFromAllAgents` 一键扫描所有 agent(同名跨 agent 去重、同目录按 realpath 只扫一次、未安装/无目录记 skippedAgents 不报错);桌面 App 启动时(startServer)自动做用户级收养,打开即在技能库看到本机已配置的 skills。
- **AI 技能推荐**(`src/core/ai.ts`):填开发需求,模型(OpenAI 兼容 chat/completions)读本地技能库给出初步推荐供勾选绑定,**新建项目弹窗与项目详情页均可多次调用**;同时**联网搜 GitHub**——模型输出 githubKeywords(没给则用需求里的英文词兜底),按 `topic:agent-skills <关键词>` 搜仓库(复用 recommend 的 24h 缓存),去重、排除已入库、按 star 降序,可一键安装入库;本地与联网两路成败互相隔离(库空跳过模型只走联网;模型挂了仍有 GitHub 结果);配置存 `ai.json`(baseUrl/model/apiKey,预设 Kimi/DeepSeek/OpenAI/OpenRouter,baseUrl 可填中转站);未配置 key/断网/解析失败一律降级为 `{ items: [], github: [], message }` 不抛异常;CLI(`ssw ai`)、REST(`/api/ai/*`)、桌面 GUI(新建项目弹窗 + 项目详情「AI 推荐」区 + 设置弹窗)、TUI(i 键)均已接入。
- **快照回滚**:每次 apply 前在 `snapshots/<projectId>/` 建快照(全局共享用 `snapshots/__global__/`),每项目保留最近 5 份,`rollback` 逆序还原最近一次(skills 与 MCP 同一份快照,一起还原)。
- **热度排序选配**(`src/core/rank.ts`):给项目/全局共享选技能时,常用的排前面。三个信号加权:使用次数(绑定即计——`registry.markSkillsUsed` 挂在 updateProject/updateGlobal 的技能集差集上,只增不减,每次 +10)> 项目分类匹配(技术栈 + 项目名分词命中 skill 的 name/description/tags,每个 +6;`projectRankContext` 复用 recommend 的检测/分词口径)> 仓库 stars(安装/更新时 `fetchRepoStars` 采集,软失败;log10 压量纲 ×4)。`rankSkills` 稳定降序;REST `GET /api/skills?rank=1[&forProject=id]`(不带 rank 保持注册表原顺序,向后兼容)、GUI 两个"从库中添加"弹窗、AI 推荐载荷(stars/uses 作相关度 tie-break)均已接入;CLI `skill list` 与 TUI 技能库视图带 ★/用N 热度标记;重装/更新 skill 时 upsert 保留 useCount/stars 统计。
- **自动更新系统**(`src/core/update.ts`):对照 GitHub Releases 最新 release 检查新版本——版本比较只看 X.Y.Z 数字段(解析不了按更旧,坏 tag 不误报),6h 磁盘缓存(`cache/update-latest.json`)+ 并发在途去重,断网/限流降级 `{ ok:false, message }` 不抛异常。`pickAsset` 按平台挑安装包(win→Setup*.exe / mac→按 arch 匹配 dmg / linux→AppImage);下载流式写 `.part` 再原子改名落盘 `<SSW_HOME>/downloads/`(置可执行位),进度并入 `/api/progress`,同文件已下载幂等跳过。配置存 `update.json`(autoCheck 默认开 / autoDownload 默认关 / skillsAutoCheck 默认开 / skillsCheckIntervalHours 默认 6,收敛 1-168);桌面 App 启动时按配置自动检查(开自动下载则后台拉包),全静默。CLI(`ssw update`)、REST(`/api/update/*`)、桌面 GUI(设置弹窗 + 侧栏横幅)、TUI(U 键)均已接入。
- **技能库更新系统**(library.ts `checkLibraryUpdates` / `applyLibraryUpdates` / `getLastLibraryUpdates`):检查 github 来源 skills 的上游更新并一键更新。`groupGithubRepos` 按 owner/repo 分组(整仓一次 clone、多 skill 共享,仓库目录 `library/github__owner__repo`);逐仓 `git fetch --quiet` 后 `rev-list --count HEAD..@{u}` 得落后提交数(浅克隆无上游信息时兜底 `rev-parse` 比 sha);inflight 并发去重,单仓失败只记该仓 error、整体失败降级 `{ ok:false, message }` 不抛;结果存内存态。`applyLibraryUpdates(repoIds?)` 逐 skill 走 `updateSkill`(保留 useCount/stars),成功仓库即时把内存态 behind 清零。定时调度挂在 serve.ts(listen 后 15s 首查 + 按 `skillsCheckIntervalHours` 间隔,`setInterval(...).unref()`,全静默);REST(`/api/skills/updates*`)、CLI(`skill update --check`)、TUI(技能库 U 键)、桌面 GUI(技能库页提示条 + 一键更新 + 设置开关)均已接入。

两个前端共享同一个 TypeScript 核心引擎(`src/core/`),也共享同一份磁盘状态(`SSW_HOME`):

1. **Electron 桌面 App**:主进程内**进程内启动** Express(`127.0.0.1` + 随机空闲端口),不依赖外部 node 进程;单实例锁,窗口全关即退出;窗口加载 `public/` 单页应用。
2. **CLI(`ssw`,别名 `skills`)**:commander 实现,子命令纯命令行非交互,适合服务器;子命令完整映射 core 能力,全局 `--json` 输出。**不带参数启动(TTY 下)进入交互式终端面板**(`src/tui.ts`,零依赖:stdin raw 模式 + ANSI 渲染);非 TTY 裸跑打印帮助。

## 技术栈

- **语言/运行时**:TypeScript 5.7(strict)、Node.js(ESM,`"type": "module"`,`module: NodeNext`;CLI 单文件分发目标 Node ≥ 18)。
- **运行时依赖仅两个**:`express`(REST API + 静态托管)、`commander`(CLI)。不要轻率新增运行时依赖。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chongrong1234/Skills_switchtool](https://github.com/Chongrong1234/Skills_switchtool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
