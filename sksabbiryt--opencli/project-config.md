---
trigger: always_on
description: OpenCLI — Make any website or Electron App your CLI. Zero risk, AI-powered, reuse Chrome login.
---


# OpenCLI

> Make any website or Electron App your CLI. Reuse Chrome login, zero risk, AI-powered discovery.

> [!CAUTION]
> **AI Agent 必读：创建或修改任何适配器之前，你必须先阅读 [CLI-EXPLORER.md](./CLI-EXPLORER.md)！**
> 该文档包含完整的 API 发现工作流（必须使用浏览器探索）、5 级认证策略决策树、平台 SDK 速查表、`tap` 步骤调试流程、分页 API 模板、级联请求模式、以及常见陷阱。
> **本文件（SKILL.md）仅提供命令参考和简化模板，不足以正确开发适配器。**

> [!IMPORTANT]
> 创建或修改 adapter 时，再额外遵守 3 条收口规则：
> 1. 主参数优先用 positional arg，不要把 `query` / `id` / `url` 默认做成 `--query` / `--id` / `--url`
> 2. 预期中的 adapter 失败优先抛 `CliError` 子类，不要直接 throw 原始 `Error`
> 3. 新增 adapter 或新增用户可发现命令时，同步更新 adapter docs、`docs/adapters/index.md`、sidebar，以及 README/README.zh-CN 中受影响的入口

## Install & Run

```bash
# npm global install (recommended)
npm install -g @jackwener/opencli
opencli <command>

# Or from source
cd ~/code/opencli && npm install
npx tsx src/main.ts <command>

# Update to latest
npm update -g @jackwener/opencli
```

## Prerequisites

Browser commands require:
1. Chrome browser running **(logged into target sites)**
2. **opencli Browser Bridge** Chrome extension installed (load `extension/` as unpacked in `chrome://extensions`)
3. No further setup needed — the daemon auto-starts on first browser command

> **Note**: You must be logged into the target website in Chrome before running commands. Tabs opened during command execution are auto-closed afterwards.

Public API commands (`hackernews`, `v2ex`) need no browser.

## Commands Reference

### Data Commands

```bash
# Bilibili (browser)
opencli bilibili hot --limit 10          # B站热门视频
opencli bilibili search "rust"            # 搜索视频 (query positional)
opencli bilibili me                       # 我的信息
opencli bilibili favorite                 # 我的收藏
opencli bilibili history --limit 20       # 观看历史
opencli bilibili feed --limit 10          # 动态时间线
opencli bilibili user-videos --uid 12345  # 用户投稿
opencli bilibili subtitle --bvid BV1xxx   # 获取视频字幕 (支持 --lang zh-CN)
opencli bilibili dynamic --limit 10       # 动态
opencli bilibili ranking --limit 10       # 排行榜
opencli bilibili following --limit 20     # 我的关注列表 (支持 --uid 查看他人)

# 知乎 (browser)
opencli zhihu hot --limit 10             # 知乎热榜
opencli zhihu search "AI"                # 搜索 (query positional)
opencli zhihu question 34816524            # 问题详情和回答 (id positional)

# 小红书 (browser)
opencli xiaohongshu search "美食"           # 搜索笔记 (query positional)
opencli xiaohongshu notifications             # 通知（mentions/likes/connections）
opencli xiaohongshu feed --limit 10           # 推荐 Feed
opencli xiaohongshu user xxx               # 用户主页 (id positional)
opencli xiaohongshu creator-notes --limit 10   # 创作者笔记列表
opencli xiaohongshu creator-note-detail --note-id xxx  # 笔记详情
opencli xiaohongshu creator-notes-summary      # 笔记数据概览
opencli xiaohongshu creator-profile            # 创作者资料
opencli xiaohongshu creator-stats              # 创作者数据统计

# 雪球 Xueqiu (browser)
opencli xueqiu hot-stock --limit 10      # 雪球热门股票榜
opencli xueqiu stock --symbol SH600519   # 查看股票实时行情
opencli xueqiu watchlist                 # 获取自选股/持仓列表
opencli xueqiu feed                      # 我的关注 timeline
opencli xueqiu hot --limit 10            # 雪球热榜
opencli xueqiu search "特斯拉"            # 搜索 (query positional)
opencli xueqiu earnings-date SH600519    # 股票财报发布日期 (symbol positional)
opencli xueqiu fund-holdings             # 蛋卷基金持仓明细 (支持 --account 过滤)
opencli xueqiu fund-snapshot             # 蛋卷基金快照（总资产、子账户、持仓）

# GitHub (via gh External CLI)
opencli gh repo list                     # 列出仓库 (passthrough to gh)
opencli gh pr list --limit 5             # PR 列表
opencli gh issue list                    # Issue 列表

# Twitter/X (browser)
opencli twitter trending --limit 10      # 热门话题
opencli twitter bookmarks --limit 20     # 获取收藏的书签推文
opencli twitter search "AI"              # 搜索推文 (query positional)
opencli twitter profile elonmusk         # 用户资料
opencli twitter timeline --limit 20      # 时间线
opencli twitter thread 1234567890        # 推文 thread（原文 + 回复）
opencli twitter article 1891511252174299446 # 推文长文内容
opencli twitter follow elonmusk          # 关注用户
opencli twitter unfollow elonmusk        # 取消关注
opencli twitter bookmark https://x.com/... # 收藏推文
opencli twitter unbookmark https://x.com/... # 取消收藏
opencli twitter post "Hello world"       # 发布推文 (text positional)
opencli twitter like https://x.com/...   # 点赞推文 (url positional)
opencli twitter reply https://x.com/... "Nice!" # 回复推文 (url + text positional)
opencli twitter delete https://x.com/... # 删除推文 (url positional)
opencli twitter block elonmusk           # 屏蔽用户 (username positional)
opencli twitter unblock elonmusk         # 取消屏蔽 (username positional)
opencli twitter followers elonmusk       # 用户的粉丝列表 (user positional)
opencli twitter following elonmusk       # 用户的关注列表 (user positional)
opencli twitter notifications --limit 20 # 通知列表
opencli twitter hide-reply https://x.com/... # 隐藏回复 (url positional)
opencli twitter download elonmusk        # 下载用户媒体 (username positional, 支持 --tweet-url)
opencli twitter accept "群,微信"          # 自动接受含关键词的 DM 请求 (query positional)
opencli twitter reply-dm "消息内容"       # 批量回复 DM (text positional)

# Reddit (browser)
opencli reddit hot --limit 10            # 热门帖子
opencli reddit hot --subreddit programming  # 指定子版块

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sksabbiryt/opencli](https://github.com/sksabbiryt/opencli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
