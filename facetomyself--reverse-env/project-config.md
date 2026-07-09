---
trigger: always_on
description: 本文件与 `CLAUDE.md` 并存，功能对等。Codex 在 `D:\reverse_ENV` 及子目录下工作时自动加载。
---

# 逆向工程环境配置与技能仓库（Codex）

本文件与 `CLAUDE.md` 并存，功能对等。Codex 在 `D:\reverse_ENV` 及子目录下工作时自动加载。

> MCP 项目配置位于 `.mcp.json` 与 `.codex/config.toml`：ida-multi-mcp、ruyi-mcp。`jadx-ai-mcp`、`js-reverse-mcp`、`reqable`、`first-mcp` 统一按需手动启用，默认不自动初始化。Codex 用户级 `~/.codex/config.toml` 只保留 provider、features、plugins、trust 等个人默认，不放 `D:\reverse_ENV` 专属 MCP。搜索类能力（`search-layer` / `github-solution-research`）属于全局分级策略，不放入项目 `.mcp.json`；Claude 全局环境已有 `search-layer`，Codex 侧已迁移本地 skill 副本到 `~/.codex/skills/search-layer`，并完成 `search.py --mode deep --intent resource` 三源 smoke test（Exa + Tavily + Grok）。

## 任务前强制检查

| # | 检查项 | 动作 |
|---|--------|------|
| 1 | **WebFetch 封禁** | 需要取 URL 内容？→ 禁止 WebFetch，走 `search-layer` / `github-solution-research` / 浏览器方案 |
| 2 | **知识库检索** | 新项目/新分析任务？→ 查 `docs/article-index.md` 按主题/技术标签检索现成文章 |
| 3 | **搜索** | 新任务/新问题？→ `search-layer` → `github-solution-research`，确认无现成方案再动手 |
| 4 | **Git 状态** | `git status --short --branch` |
| 5 | **远端同步** | `git remote -v`；涉及 PR/远端时 `git fetch origin && git log --oneline origin/main..HEAD` |

> WebFetch 硬封禁：即使 URL 看起来可访问，也不得直接调用 WebFetch。优先用全局 `search-layer` (client-native search + Exa + Tavily + Grok)、`github-solution-research` (GitHub)、浏览器 MCP (需登录/JS 渲染)。Claude / Codex 均已有本地 `search-layer` skill；若某个源不可用，需明确标注可用替代路径。**此规则优先于所有其他工具选择逻辑。**

## 核心约束

**所有 skill、MCP、Python venv、工具依赖均安装在 `D:\reverse_ENV\` 内，不得污染系统全局。**

- 项目 skill 源目录: `skill\<name>\`；Codex repo-scope 发现入口: `.agents\skills\<name>\` 薄封装。
- `.agents\skills\` 只保留 frontmatter 与源 skill 路由说明，不复制脚本/参考资料/流程正文；真实维护仍在 `skill\<name>\`。
- `.agents\skills\<name>\SKILL.md` 必须与 `skill\<name>\SKILL.md` 一一对应，只允许指向源 skill；新增/删除/重命名 skill 时两侧同步。
- venv: `.venv\` ｜ JDK: `tools\jdk\` ｜ Node: `tools\node\`
- NDK r29: `tools\android-ndk\` ｜ Rust: `%USERPROFILE%\.cargo\`
- IDA Pro 9.3: `resource\portable_win\` ｜ MCP 配置: `.mcp.json` + `.codex\config.toml`（Codex 项目层）+ `~/.codex/config.toml`（Codex 用户默认）
- **所有逆向项目在 `workspace\<项目名>\` 下起新文件夹**。产出物均落地到对应项目目录。
- **待分析二进制文件**（`.dll`, `.so`, `.exe`, `.bin` 等）**必须先放入 `workspace\<项目名>\`**，再打开 IDA/radare2。禁止将二进制文件直接放在 `workspace\` 根目录。
- **IDA 数据库文件**（`.id0`, `.id1`, `.id2`, `.nam`, `.til`, `.i64`）由 IDA 在二进制文件所在目录自动生成。确保二进制文件在项目子目录内，即可避免 IDA 产物污染根目录。
- **抓包流量文件**（`*.flow`, `*.pcap`, `*.har`）统一放在 `workspace\<项目名>\` 下。
- **`storage\` 存放可复用的大文件**（安装包、SDK、ISO 等），内容不纳入 Git。

### AI 协作子约束

> 详见 `docs/AI开发规范.md` 和 `docs/Git与提交规范.md`

**操作纪律：**
1. **不得凭记忆** — 修改文件前 Read 实际内容，不基于摘要操作。
2. **先确认、再动手** — 确认当前目录、文件存在、工具可用。
3. **改动闭环** — 改脚本 → 同步 CLAUDE.md + AGENTS.md；改工具路径 → 同步 skill 文档；加项目 MCP → 同步 `.mcp.json` + `.codex/config.toml`；加 Codex 全局 MCP → 同步 `~/.codex/config.toml`。
4. **禁止猜测** — 工具安装、命令执行必须有真实输出为证。

**编码：**
5. UTF-8 + LF 新文件；已有 BOM 保留 BOM；中文文件防乱码。
6. 路径全部绝对化，不依赖 PATH。
7. **禁止滥用 emoji** — 文档、代码注释、CLAUDE.md/AGENTS.md、skill 文件、提交信息中不使用 emoji 作为项目符号或装饰。用纯文本标记（`-`/`*`/`#`）替代。

**渐进式披露：**
8. **先侦察、后深挖** — 从轻量 triage 开始，根据 marker 决定深度。不得一上来全量分析/盲目 Hook。
9. **证据优先** — 每个结论必须有可追溯证据。未经证实的标注"待验证"。
10. **能力匹配复杂度** — L1 便携 → L2 上下文 → L3 运行时 → L4 triage-only。不假装能完整还原 WASM/VM。

**产出规范：**
11. 每次分析产出三件套：`report.md` + `findings.json` + `triage.md`（模板: `skill/reverse-coordinator/templates/`）。
12. **审查门** — 产出前自检：claim 有证据？triage 已标注？敏感数据已脱敏？
13. **不得假装** — 不对 L4 目标声称"已完整复现"。

**修改闭环：**
14. 自检：CLAUDE.md/AGENTS.md 路径一致？`.mcp.json` / `.codex/config.toml` / `~/.codex/config.toml` 合法？临时文件已清理？敏感数据已脱敏？

## 仓库入口

| 想看什么 | 去哪里 |
|----------|--------|
| **完整目录树** | 各目录 README: `skill/README.md`, `tools/README.md`, `mcp/README.md`, `docs/README.md`, `workspace/README.md`, `resource/README.md`, `storage/README.md` |
| 工具版本与路径 | `tools/README.md` + `docs/工具与环境.md` |
| App 逆向环境规划 | `docs/App逆向环境规划.md` |
| MCP 服务配置详情 | `mcp/README.md` + `docs/MCP服务详情.md` |
| Skill 清单 | `skill/README.md` |
| Codex repo-scope skill 入口 | `.agents/skills/README.md` |
| 工作流与深度等级 | `docs/逆向工作流详解.md` |
| Web 逆向架构分析 | `docs/Web逆向架构分析.md` |
| ruyi-mcp 引导方案 | `docs/ruyi-mcp-引导方案.md` |
| 脚本使用说明 | `docs/脚本参考.md` |
| AI 协作开发规范 | `docs/AI开发规范.md` |
| Git 操作规范 | `docs/Git与提交规范.md` |
| **逆向知识库索引** | `docs/article-index.md` — 按主题/技术标签检索跨项目可复用分析文章 |
| 逆向知识库文章 | `article/` — 协议分析/反检测/签名算法/加固绕过/Native分析/Web逆向 |

## 任务前知识库检索（硬纪律）

**新项目/新分析任务启动时，必须先查 `docs/article-index.md`**，确认是否有现成的同主题/同厂商/同技术栈分析文章可复用。跳过 → 违规。

| 场景 | 检索方向 |
|------|---------|
| 遇到新协议 | 查 `article/protocols/` + 标签「协议」 |
| 遇到签名/加密 | 查 `article/signature-algorithms/` + 标签「密码学」 |
| 遇到反调试/加固 | 查 `article/packing-bypass/` + 标签「反检测/对抗」 |
| 遇到 Web 框架/打包 | 查 `article/web-reverse/` + 标签「Webpack」 |
| 遇到风控/验证码 | 查 `article/anti-detection/` + 标签「WAF」「设备指纹」 |

## Skill 速查

| Skill | 场景 | 何时用 |
|-------|------|--------|
| `reverse-coordinator` | **元 skill** | 未指定工具时优先——分类→路由→编排→交付 |
| `apk-reverse` | Android APK | jadx/apktool/frida/adb + 指纹/脱壳/Kotlin类名恢复/API提取 |
| `ida-reverse` | PE/ELF/DLL/SO | IDA Pro 深度分析 + IDAPython速查/符号恢复/结构体恢复 |
| `ruyi-reverse` | Web JS — 统一编排器 | 7 能力模块 x 深浅两级，按任务主动组合。**唯一入口** |
| `web-env-patcher` | Web JS Node 补环境 | 接在 ruyi/js-reverse 取证后：隔离 runtime、cURL/HAR 检查、Trace API 覆盖矩阵、fixtures、TLS 门禁 |
| `proxy-usage` | 代理统一管理 | 快代理 + Cliproxy 双供应商 — 选型→提取→验证→注入 |
| `radare2` | 通用二进制 | CLI 快速侦察/反汇编/patch |
| `native-reverse` | Android Native .so 反检测/绕过 | syscall 定位→dump/fix→IDA→patch→验证 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Facetomyself/reverse_ENV](https://github.com/Facetomyself/reverse_ENV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
