---
trigger: always_on
description: feedgrab 是一个万能内容抓取器，从任意平台抓取内容并输出为 Obsidian 兼容的结构化 Markdown。
---

# feedgrab 项目指令

## 项目概述

feedgrab 是一个万能内容抓取器，从任意平台抓取内容并输出为 Obsidian 兼容的结构化 Markdown。

- **仓库**：https://github.com/iBigQiang/feedgrab
- **作者**：[@iBigQiang](https://github.com/iBigQiang)（强子手记）
- **当前版本**：v0.15.1
- **Python**：≥3.10
- **许可证**：MIT

### 项目来源

feedgrab 由两个项目融合升级而来：
- **[x-reader](https://github.com/runesleo/x-reader)**（@runes_leo）— 提供多平台架构、CLI、MCP 服务器
- **[baoyu-danger-x-to-markdown](https://github.com/JimLiu/baoyu-skills)**（@dotey 宝玉）— 提供逆向工程的 X/Twitter GraphQL 深度抓取能力

### 三层架构

| 层级 | 功能 | 入口 |
|------|------|------|
| Python CLI/库 | 基础内容抓取 + 统一数据结构 | `feedgrab <url>` |
| Codex 技能 | 视频转录 + AI 分析 | `skills/video/` `skills/analyzer/` |
| MCP 服务器 | 将抓取能力暴露为 MCP 工具 | `mcp_server.py` |

### 支持的平台

| 平台 | 抓取方式 |
|------|---------|
| X/Twitter | GraphQL → FxTwitter → Syndication → oEmbed → Jina → Playwright（六级兜底） |
| 小红书 | API (xhshow) → Pinia Store 注入 → Jina → Playwright 深度抓取（单篇 + 作者批量 + 搜索批量 + xhs-so 搜索） |
| YouTube | InnerTube API 字幕（零依赖零 quota）→ yt-dlp 字幕 → Groq Whisper 转录 + YouTube Data API v3 搜索 + yt-dlp 下载 |
| B站 | API |
| 微信公众号 | Playwright WeChat JS 提取 → Jina 兜底（单篇 + markdownify 富文本）/ 搜狗搜索（关键词搜索）/ MP 后台 API（按账号批量）/ 专辑批量（`mpweixin-zhuanji`） |
| GitHub | REST API（仓库元数据 + 中文 README 优先 + 摘要提取） |
| 飞书/Lark | Open API → Playwright PageMain Block 树 → Jina（单篇 + 知识库批量 + 嵌入表格 + 图片下载） |
| 金山文档/KDocs | Playwright ProseMirror DOM 提取（虚拟滚动 + 代码块 + 图片 shapes API + CDP 直连） |
| 有道云笔记 | JSON API（零依赖）→ Playwright iframe DOM → Jina（单篇 + 图片下载） |
| 知乎 | API v4 → Playwright CDP/DOM → Jina（单篇问答前 3 楼 + 专栏文章 + 关键词搜索 `zhihu-so`） |
| Telegram | Telethon |
| RSS | feedparser |
| 任意网页 | Jina 兜底 |

## 语言规范

- 所有对话、文档内容一律使用**中文**
- 代码注释可以用英文
- Git commit message 使用英文前缀（feat/fix/docs/chore）+ 英文描述

## 开发工作流

完成功能开发并测试通过后，执行以下收尾流程：

1. **更新 DEVLOG.md** — 在文件顶部（第一个 `---` 分隔线之前）新增版本条目
2. **更新 README.md** — 同步新功能的使用说明、配置项、架构图等
3. **提交代码** — 使用 conventional commit 格式
4. **推送到 GitHub** — `git push origin main`

> 提示：可以使用 `/ship` 命令一键完成上述收尾流程。

## Codex 快捷命令约定

为兼容 Codex，本项目将 Claude Code 的命令文件映射为“触发词 + 脚本”模式：

- 当用户输入 `/newup` 时，视为执行项目启动预热：
  - 优先运行 `scripts/newup.ps1`
  - 同时读取 `CLAUDE.md`、`AGENTS.md`、`DEVLOG.md`、`README.md`
  - 输出当前版本、最近一次迭代标题、项目上下文摘要和 AGENTS 规则预览，然后再继续当前任务
- 当用户输入 `/ship` 时，视为执行收尾检查流程：
  - 优先运行 `scripts/ship.ps1`
  - 查看 `git status --short`、`git diff --stat`
  - 检查 `DEVLOG.md`、`README.md`、`README_EN.md`、`AGENTS.md` 是否存在
  - 基于检查结果继续执行文档更新、提交、推送等后续动作
- `.claude/commands/newup.md` 与 `.claude/commands/ship.md` 保留给 Claude Code 使用，Codex 不直接读取它们作为 slash 命令入口

## 版本号规范

- 主要新功能：递增次版本号（如 v0.2.9 → v0.3.0）
- 小功能/修复：递增补丁号（如 v0.3.0 → v0.3.1）
- 版本号记录在 DEVLOG.md 的条目标题中

## 核心架构

```
feedgrab/
├── feedgrab/                  # Python 包
│   ├── cli.py                 # CLI 入口（命令路由 + feedgrab setup 引导 + feedgrab clip 剪贴板抓取）
│   ├── config.py              # 集中配置（路径、开关、get_user_agent() + get_stealth_headers()）
│   ├── reader.py              # URL 调度器（UniversalReader — 平台检测 + 路由 + URL 规范化）
│   ├── schema.py              # 统一数据模型（UnifiedContent）
│   ├── login.py               # 浏览器登录管理器（+ CDP Cookie 提取）
│   ├── fetchers/
│   │   ├── jina.py            # Jina Reader（万能兜底）
│   │   ├── browser.py         # 隐身浏览器引擎（patchright Tier 1 → playwright Tier 3 + stealth flags）
│   │   ├── bilibili.py        # B站 API
│   │   ├── youtube.py         # yt-dlp 字幕提取 + API 优先元数据
│   │   ├── youtube_search.py  # YouTube Data API v3 搜索 + yt-dlp 下载（视频/音频/字幕）
│   │   ├── github.py          # GitHub REST API（仓库元数据 + 中文 README 优先）
│   │   ├── rss.py             # RSS 解析
│   │   ├── telegram.py        # Telegram 频道
│   │   ├── twitter.py         # X/Twitter 六级兜底调度器
│   │   ├── twitter_cookies.py # Cookie 多源管理 + 多账号轮换（429 自动切换）
│   │   ├── twitter_fxtwitter.py # FxTwitter API 客户端（Tier 0.3 兜底 + circuit breaker）
│   │   ├── twitter_graphql.py # GraphQL API（TweetDetail/UserTweets/Bookmarks/SearchTimeline/动态queryId + x-client-transaction-id）
│   │   ├── twitter_thread.py  # 线程重建 + 评论分类
│   │   ├── twitter_bookmarks.py  # 书签批量抓取
│   │   ├── twitter_user_tweets.py # 用户推文批量抓取
│   │   ├── twitter_list_tweets.py # List 列表批量抓取（按天数过滤+会话去重）
│   │   ├── twitter_search_tweets.py # 浏览器搜索补充抓取（突破 UserTweets 800 条限制）
│   │   ├── twitter_keyword_search.py # 关键词搜索（x-so 命令，纯 GraphQL + 互动排序表格）
│   │   ├── twitter_api.py     # TwitterAPI.io 付费 API 客户端
│   │   ├── twitter_api_user_tweets.py # 付费 API 批量抓取（max_id 分页+断点续传+智能直保）
│   │   ├── twitter_markdown.py   # Markdown 渲染器
│   │   ├── wechat.py          # Playwright → Jina（Browser 优先）
│   │   ├── wechat_search.py   # 搜狗微信搜索（关键词 → 文章发现 + 抓取）
│   │   ├── mpweixin_account.py # 微信公众号按账号批量（MP 后台 API + 断点续传）
│   │   ├── mpweixin_album.py  # 微信公众号专辑批量（mpweixin-zhuanji 命令 + 断点续传）
│   │   ├── xhs.py             # 小红书单篇（API → Pinia → Jina → Playwright 四级兜底）
│   │   ├── xhs_api.py         # 小红书 API 客户端（xhshow 签名 + 评论 + xsec_token 缓存）
│   │   ├── xhs_pinia.py       # 小红书 Pinia Store 注入（浏览器原生请求兜底，CDP 优先 + Launch 降级）
│   │   ├── xhs_user_notes.py  # 小红书作者批量（API → Pinia → 浏览器三层策略）
│   │   ├── xhs_search_notes.py # 小红书搜索批量 + xhs-so 关键词搜索（含 Pinia 兜底）
│   │   ├── feishu.py          # 飞书单篇（Open API → Playwright PageMain → Jina + Block→MD + Sheet Protobuf 解码 + 图片下载）
│   │   ├── feishu_wiki.py     # 飞书知识库批量（Open API 递归 + Playwright 兜底 + 断点续传）
│   │   ├── kdocs.py           # 金山文档单篇（Playwright ProseMirror DOM + 虚拟滚动 + CDP 直连 + 图片 shapes API）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iBigQiang/feedgrab](https://github.com/iBigQiang/feedgrab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
