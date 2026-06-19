---
trigger: always_on
description: Automatically collect and archive content from shared links in group chats. When a user shares a link (WeChat articles, Feishu docs, web pages, etc.) in any group chat and asks to archive/collect/save it, this skill triggers to fetch the content, create a Feishu document, and update the knowledge base table. Use when: (1) User shares a link and asks to "收录/转存/保存" content, (2) Need to archive web content to Feishu docs, (3) Building a personal knowledge base from shared links, (4) Organizing lear
---


# Content Collector - 链接内容自动收录技能

## Overview

This skill enables automatic collection and archiving of content from shared links into a structured knowledge base.

**Core Workflow:**
```
Detect Link → Fetch Content → Extract Images → Upload Images to Feishu → Create Feishu Doc → Update Table
```

## When to Use

### 模式1：主动触发（显式关键词）
当用户消息包含以下**触发词**时，立即执行收录：
- "收录" / "转存" / "保存" / "存档" / "存一下" / "归档" / "备份" / "收藏"
- "存到知识库" / "加入知识库" / "转飞书"

**示例：**
- "这个链接收录一下"
- "存到知识库"
- "转存这篇教程"

### 模式2：静默收录（自动检测）
在**群聊场景**中，自动检测以下链接并静默收录：
- 飞书文档/表格/Wiki（feishu.cn）
- 微信公众号文章（mp.weixin.qq.com）
- 技术博客/教程站点
- 知识分享类链接

**静默收录条件：**
1. 消息来自群聊（非私聊）
2. 消息包含可识别的知识类链接
3. 用户没有明确拒绝的意图

**两种模式优先级：**
```
检测到主动触发词 → 立即收录（显式模式）
未检测到触发词但检测到链接 → 静默收录（隐式模式）
```

## Supported Link Types

| Type | Example | Fetch Method |
|------|---------|--------------|
| WeChat Article | `https://mp.weixin.qq.com/s/xxx` | kimi_fetch |
| Feishu Doc | `https://xxx.feishu.cn/docx/xxx` | feishu_fetch_doc |
| Feishu Wiki | `https://xxx.feishu.cn/wiki/xxx` | feishu_fetch_doc |
| Web Page | General URLs | kimi_fetch / web_fetch |

## Supported Image Sources

| Source | Example | Priority | Notes |
|--------|---------|----------|-------|
| Markdown image | `![alt](https://xx/image.png)` | High | 直接替换为飞书 image_key |
| HTML `<img src>` | `<img src="/assets/a.png">` | High | 相对路径需转绝对路径 |
| Lazy-load image | `data-src`, `data-original` | Medium | 常见于公众号/博客懒加载 |
| `srcset` candidate | `srcset="a 1x, b 2x"` | Medium | 优先选择清晰度更高的候选图 |
| Feishu file token | `boxcn...` / `img_v3_...` | High | 需要走飞书素材下载后再上传 |

## Global Availability (全局可用配置)

**生效范围：所有用户、所有群聊**

本技能已配置为全局可用，支持以下对象：

| 对象类型 | 支持状态 | 说明 |
|---------|---------|------|
| **所有用户** | ✅ 可用 | 任何用户分享的链接均可被收录 |
| **所有群聊** | ✅ 可用 | 支持技能中心群、养虾群、学习群等所有群组 |
| **私聊消息** | ✅ 可用 | 用户私信分享链接也可触发收录 |
| **多渠道** | ✅ 可用 | 飞书、其他渠道统一支持 |

**权限说明：**
- 任何用户均可触发收录（无需管理员权限）
- 收录的文档统一存储到指定的知识库目录
- 所有用户均可查看已收录的文档

---

## Installation & Permission Check (安装与权限检查)

在正式使用本技能前，系统必须自动或引导用户完成以下权限校验，以确保流程不中断：

### 1. 飞书权限清单
| 权限项 | 验证工具 | 目的 |
|-------|---------|------|
| **OAuth 授权** | `feishu_oauth` | 获取操作飞书文档和表格的用户凭证 |
| **知识库写入权限** | `feishu_create_doc` | 确保能在指定的 Space ID 下创建节点 |
| **多维表格编辑权限** | `feishu_bitable_app_table_record` | 确保能向指定的 app_token 写入记录 |
| **图片上传权限** | `feishu_im_bot_upload` | 允许将本地图片同步至飞书素材库 |

### 2. 预检流程 (Pre-flight Check)
每次“安装”或配置更新后，执行以下检查：
1. **验证 Space ID 可访问性**：尝试在指定目录下获取节点列表。
2. **验证 Table 结构**：检查 `关键词`、`原链接`、`图片数量`、`图片处理状态` 等字段是否存在（后两者可选）。
3. **静默测试**：如果权限不足，立即通过 `feishu_oauth` 弹出授权引导，而非在执行收录时报错。

---

## Configuration

Before using, ensure these are configured in MEMORY.md:

```markdown
## Content Collector Config
- **Knowledge Base Table**: `[Your Bitable App Token]` (Bitable app_token)
- **Table URL**: [Your Bitable Table URL]
- **Default Table ID**: `[Your Table ID]` (will auto-detect if available)
- **Knowledge Base Space ID**: `[Your Space ID]` (所有文档创建在此知识库下)
- **Knowledge Base URL**: [Your Knowledge Base Homepage URL]
- **Content Categories**: 技术教程, 实战案例, 产品文档, 学习笔记
- **Global Access**: 所有用户可用，所有群聊可用
- **Image Fetch Mode**: `all` / `cover_only`（默认 `all`）
- **Image Max Count**: `20`（单篇文档最多处理图片数）
- **Image Max Size MB**: `10`（单图超过阈值则跳过）
- **Image Timeout Sec**: `20`（下载超时）
- **Image Allowed Types**: `jpg,png,gif,webp`
- **Image Fallback**: `keep_original_link=true`
```

**Note**: 
1. This skill updates ONLY the configured knowledge base table. Do not create or update any other tables.
2. **All created documents must be saved under the designated Knowledge Base** using wiki_node parameter.
3. **Global Access**: 所有用户、所有群聊均可使用本技能，收录的文档对全员可见。
4. 图片抓取默认开启；若用户明确要求“纯文字收录”，可跳过图片处理。

---

## 📚 知识库文档存储规则（必遵守）

所有收录的文档必须按照以下规则分类存储到知识库对应目录：

### 知识库目录结构

请参考各项目或团队定义的知识库标准目录结构进行存储。收录的文档通常存放在“素材”或“归档”类目录下。

### 文档分类映射规则

| 内容分类 | 存储目录 (wiki_node) | 命名前缀 | 示例 |
|----------|---------------------|----------|------|
| 技术教程 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 📖 | 📖 [标题] |
| 实战案例 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 🛠️ | 🛠️ [标题] |
| 产品文档 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 📄 | 📄 [标题] |
| 学习笔记 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 💡 | 💡 [标题] |
| 热点资讯 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 🔥 | 🔥 [标题] |
| 设计技能 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 🎨 | 🎨 [标题] |
| 工具推荐 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 🔧 | 🔧 [标题] |
| 训练营 | `F9pFw9dxTiXmpsk5bNlco704nag` (内容文档) | 🎓 | 🎓 [标题] |

### 文档命名规范

```
[Emoji前缀] [原标题] | 收录日期

示例：
📖 OpenClaw保姆级教程 | 2026-03-08
🛠️ 火山方舟自动化报表案例 | 2026-03-08
🔥 GPT-5.4发布解读 | 2026-03-08
```

### 文档模板

```markdown
# [Emoji] [原标题]

> 📌 **元信息**
> - 来源：[原始来源]
> - 原文链接：[原始URL]
> - 收录时间：YYYY-MM-DD
> - 内容分类：[技术教程/实战案例/产品文档/学习笔记/热点资讯/设计技能/工具推荐/训练营]
> - 关键词：[关键词1, 关键词2, 关键词3]

---

## 📋 核心要点

[3-5条核心内容摘要]

---

## 📝 正文内容

[完整的转存内容]

---

## 🔗 相关链接

- 原文链接：[原始URL]
- 知识库索引：[素材池文档索引链接]

---

📚 **收录时间**：YYYY-MM-DD  
🏷️ **分类**：[分类名]  
🔖 **关键词**：[关键词]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuhongchen/content-collector-skill](https://github.com/wuhongchen/content-collector-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
