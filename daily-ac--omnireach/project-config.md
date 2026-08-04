---
trigger: always_on
description: This file is loaded automatically when Claude Code starts in this repo. It carries omnireach project knowledge that previously lived in user-global memory; centralizing it here lets the context follow the repo across machines and sessions.
---

# CLAUDE.md — omnireach 项目协作上下文

This file is loaded automatically when Claude Code starts in this repo. It carries omnireach project knowledge that previously lived in user-global memory; centralizing it here lets the context follow the repo across machines and sessions.

## 项目是什么

**omnireach 是项目名, 同 repo 内多个职责子命令 (subcommand 形态, 落地后再判断要不要拆 binary)**, 不是单一工具。完整"触达全网"语义由三层职责实现, 都在同 repo（**不开 sister repo**, 拍板于 2026-05-27 session）:

- **`omnireach search`** (v0.1+ 起在用): search 层 — 全网定位 metadata + URL, **不取内容**
- **`omnireach fetch`** (v0.10 起 ship 为 subcommand): fetch 层 — 给定 URL 取全文 markdown; 当前 host-aware 路径为普通网页内置 HTTP → Jina fallback，`mp.weixin.qq.com` → OpenCLI 后台临时 tab；Crawl4AI 只保留显式 opt-in
- **parse** (暂未实现): parse 层 — 视频/音频内容解析 (字幕/STT/逐帧), 真有 issue 才加

类比 `cargo` (一个 repo 下 `cargo` / `rustc` / `rustfmt` 多个 sibling binary, 共享一个项目愿景) 而不是 `git` + `git-lfs` (后者是 standalone separate repo)。**Why monorepo 而非 multirepo**: 2026-05-27 session 调研 Crawl4AI 时验证 —— 收纳 `crwl` 作为 fetch 工具不影响 search 行为, 只补齐 search 结果, 完全没有 sister repo 的耦合协调成本。一个 release 频道 + 一个 issue tracker + 一份 README 就够。

**search binary 名是否改名**: ~~曾经考虑改成 `omnisearch` 因为"reach 是触达, search 只是触达的第一步"~~ — **2026-05-27 拍板不改**。Rationale: v0.10 ship 了 `omnireach fetch` 后, `omnireach` 这个名字反而契合 umbrella 定位 —— `omnireach search` (触达 = 找) + `omnireach fetch` (触达 = 取) 都是 reach 的合理子动作, subcommand 分担具体语义, 项目名留给愿景。改名得不偿失。

本 repo 当前覆盖 search (v0.1+) + fetch (v0.10+) 两层。用户问"加 parse 能力到 omnireach 里"时按"等本 repo 加 sibling subcommand/binary, 不开 sister repo"处理; 视频解析这类大依赖立项前先看真 issue (YAGNI)。见下方"架构边界"。

## 目标用户与痛点

omnireach 是给**任何 Claude Code WebSearch 拿不到结果**的用户的工具 — 受众比"中转站"宽得多。

**WebSearch 是 server tool** (`web_search_20250305`), 真实可用性其实经过**两层 gate**, 受影响场景**不同根因不同**:

**1. 客户端 gate** (`WebSearchTool.isEnabled()` + `getAPIProvider()` 看 `CLAUDE_CODE_USE_*` env var):
- 默认 `firstParty` (没设 `CLAUDE_CODE_USE_*`, 即便只改了 `ANTHROPIC_BASE_URL` 也归这类) → tool 注册
- 显式 `CLAUDE_CODE_USE_BEDROCK=1` → 关
- 显式 `CLAUDE_CODE_USE_VERTEX=1` + Claude 4+ → 注册; 老 Claude → 关
- 显式 `CLAUDE_CODE_USE_FOUNDRY=1` → 注册

**2. 上游 server tool 实现 gate** (上游 API 必须**专门实现** `web_search_20250305` server tool — 即接 tool call → 跑搜索 → 返结果给客户端):
- 真 Anthropic (api.anthropic.com): ✓ 原生实现
- Vertex / Foundry 支持的 model: ✓ 各自 backend 实现了
- **专门支持 Claude Code 的第三方模型厂** (e.g. DeepSeek 的 Anthropic-compat 端点): ✓ 他们专门做了 server tool 处理, 接到自己的搜索后端 (data point: 2026-06-03 user 实测)
- **OpenAI 兼容中转站** (cliproxy / anyrouter 等, 把 Claude API → OpenAI Chat Completions 单纯转译): ✗ 不识 server tool 语义
- **自托管 gateway / 大部分 proxy**: ✗ 一般不专门实现

**关键 insight**: 上游 gate 看的是**"上游有没有专门做 Claude Code server tool 兼容"**, 不是"是否真 Anthropic"。专门支持 Claude Code 的厂商都实现; 单纯做 API 转译的不实现。"上游不是真 Anthropic 所以 fail" 这个 framing 是错的 (e.g. DeepSeek 不是真 Anthropic 但 WebSearch 工作)。

**分场景痛点根因**:
- 用 DeepSeek 等专门支持 Claude Code 的第三方: 两层 gate 都过, WebSearch ✓ — **omnireach 对这群人价值是补纵向源 (Twitter/小红书/微信), 不是补 search**
- OpenAI 兼容中转站用户: **上游没实现 server tool 处理** → 失败
- 显式 Bedrock 用户 / Vertex Claude 3.x: 客户端 isEnabled 直接关

**Why omnireach 不止补缺**: 即使 WebSearch 可用 (firstParty 直连), 它也搜不全 — Twitter / Reddit / 小红书 / 微信公众号 / 抖音 / B站 / TikTok 这些纵向源服务端 WebSearch 几乎都够不着。omnireach 三重价值: (1) 给客户端 gate 关掉的用户**补缺**; (2) 给上游不实现 server tool 的用户**补缺**; (3) 给 WebSearch 可用但搜不到纵向源的用户**补纵向**。CLI + MCP + Claude Code Skill 三形态。

**历史措辞修订 (2026-06-03 三轮)**:
- 第一轮 (上午): README/SKILL/本文件痛点写"中转站丢 WebSearch" — narrow
- 第二轮 (今晚 23:00): 把根因改成"Claude Code 客户端 `isEnabled()` allowlist" — 错, 因为 `isEnabled()` 看的是 `CLAUDE_CODE_USE_*` env var, 不是 `ANTHROPIC_BASE_URL`, 中转站用户实际 isEnabled=true
- 第三轮 (今晚 23:30): 拆成两层 gate, 客户端 + 上游, 上游说"必须是真 Anthropic" — 错, 因为 DeepSeek 不是真 Anthropic 但 WebSearch 工作
- 第四轮 (现在 23:44): 上游 gate 准确说法是**"是否专门实现 server tool"**, 跟"是否真 Anthropic"无关。DeepSeek 等专门支持 Claude Code 的第三方都实现; 单纯 API 转译的中转站不实现。这才是准确的。

- 位置: `~/Projects/omnireach`
- GitHub: https://github.com/Daily-AC/omnireach (Public, MIT, 归属 Daily-AC)

## 架构核心 — Umbrella + 适配器壳（v0.5 修订）

- 上游 binary 直调: `yt-dlp` (youtube) / `gh` (github) / Python `feedparser` (rss)
- OpenCLI bridge (Node + 登录态 Chrome) → reddit / twitter / xiaohongshu / tiktok / douyin
- Application services: `omnireach.service.search` + `omnireach.fetcher.fetch`，CLI 与 MCP 共用同一 envelope 和 backend 逻辑
- MCP stdio: `omnireach mcp` 零新增依赖实现 MCP 2025-06-18，暴露 `omnireach_search` / `omnireach_fetch`
- HN 直接调 Algolia Search API（无上游）
- Boosters (Tavily / Brave / Perplexity / Exa) → httpx 调付费 API，env var 检测式接入
- Agent-Reach **完全可选**（v0.5 起）: 仅作 `setup --batch` 一键引导 installer，runtime 不依赖

omnireach 自己只做: 路由 (Router) + 并发分发 (Dispatcher) + 归一 (Normalizer/Scorer) + 引导 (Wizard) + 标准化 JSON 契约 (SearchResult/SearchEnvelope, pydantic v2)。

## Tier 系统 (5 tier, sources.yml 中声明)

- ✅ `ready`: 零配置或一条 `pip install` 可用 (HN/youtube/github/rss)
- 🟡 `one_step`: 一次 OAuth/Key 配置
- 🔴 `heavy`: 装 Chrome 扩展 + 浏览器登录态 (reddit / twitter / xiaohongshu / tiktok / douyin)
- 💎 `booster`: 付费 API Key (Tavily/Brave/Perplexity/Exa)，env var 检测，结果元数据 `cost="paid"`
- 🚧 `wip`: 待重写源，sources 列表显示但不参与 auto fanout (v0.7 起暂无 wip 源)

## 架构边界 — 三层架构 (v0.7 session 拍板, 必须遵守)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Daily-AC/omnireach](https://github.com/Daily-AC/omnireach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
