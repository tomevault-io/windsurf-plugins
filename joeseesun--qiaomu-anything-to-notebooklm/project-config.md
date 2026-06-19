---
trigger: always_on
description: 多源内容智能处理器：支持微信公众号、网页、YouTube、播客（小宇宙/喜马拉雅）、PDF、Markdown等，自动上传到NotebookLM并生成播客/PPT/思维导图等多种格式。支持深度分析模式和飞书文档自动创建
---


# 多源内容 → NotebookLM 智能处理器

自动从多种来源获取内容，上传到 NotebookLM，并根据自然语言指令生成播客、PPT、思维导图等多种格式。

## 支持的内容源

### 1. 微信公众号文章
通过 MCP 服务器自动抓取微信公众号文章内容（绕过反爬虫）

### 2. 任意网页链接
支持任何公开可访问的网页（新闻、博客、文档等）

### 3. 播客（小宇宙/喜马拉雅）+ B站视频
通过 Get笔记 API 获取完整转写文本（带时间戳），支持小宇宙、喜马拉雅、B站视频等音频/视频平台

### 4. X/Twitter 帖子
通过内置代理级联（r.jina.ai → defuddle.md → agent-fetch）抓取推文内容（含长推文线程），转为 Markdown

### 5. 付费墙网站自动绕过
自动检测并绕过 NYT、WSJ、FT、Economist、Bloomberg、Medium 等 300+ 付费网站的付费墙。策略：UA 伪装（Googlebot/Bingbot）→ Referer 伪装（Google/Facebook）→ AMP 页面 → archive.today 存档

### 5. YouTube 视频
**直接传递给 NotebookLM！** NotebookLM 原生支持 YouTube 链接，会自动提取视频字幕和元数据，无需手动下载字幕或转写。禁止使用 yt-dlp 或浏览器自动化提取字幕。

### 4. Office 文档
- **Word (DOCX)** - 保留表格和格式
- **PowerPoint (PPTX)** - 提取幻灯片和备注
- **Excel (XLSX)** - 表格数据

### 5. 播客/音频平台
- **小宇宙** (xiaoyuzhoufm.com) - 通过 Get笔记 API 获取完整转写
- **喜马拉雅** (ximalaya.com) - 通过 Get笔记 API 获取完整转写
- **B站视频** (bilibili.com) - 通过 Get笔记 API 获取完整转写
- 其他音频链接 - 通过 Get笔记 API 获取转写

### 5. 电子书与文档
- **PDF** - 全文提取
- **EPUB** - 电子书全文提取
- **Markdown (.md)** - 原生支持

### 6. 图片与扫描件
- **Images** (JPEG, PNG, GIF, WebP) - OCR 识别文字
- 扫描的 PDF 文档 - OCR 提取文字

### 7. 音频文件
- **Audio** (WAV, MP3) - 语音转文字

### 8. 结构化数据
- **CSV** - 逗号分隔数据
- **JSON** - JSON 数据
- **XML** - XML 文档

### 9. 压缩包
- **ZIP** - 自动解压并处理所有支持的文件

### 10. 纯文本
直接输入或粘贴的文本内容

### 11. 搜索关键词
通过 Web Search 搜索关键词，汇总多个来源的信息

## 前置条件

### 1. 安装 wexin-read-mcp

MCP 服务器已安装在：`~/.claude/skills/qiaomu-anything-to-notebooklm/wexin-read-mcp/`

**配置 MCP**（需要手动添加到 Claude 配置文件）：

**macOS**: 编辑 `~/.claude/config.json`

```json
{
  "primaryApiKey": "any",
  "mcpServers": {
    "weixin-reader": {
      "command": "python",
      "args": [
        "/Users/joe/.claude/skills/qiaomu-anything-to-notebooklm/wexin-read-mcp/src/server.py"
      ]
    }
  }
}
```

**配置后需要重启 Claude Code。**

### 2. notebooklm 认证

首次使用前必须认证：

```bash
notebooklm login
notebooklm list  # 验证认证成功
```

## 触发方式

### 微信公众号文章
- `/qiaomu-anything-to-notebooklm [微信文章链接]`
- "把这篇微信文章传到NotebookLM"
- "把这篇微信文章生成播客"

### 网页链接
- "把这个网页做成播客 [URL]"
- "这篇文章帮我做成PPT [URL]"
- "帮我分析这个网页 [URL]"

### 播客（小宇宙/喜马拉雅/B站）
- "把这个播客生成播客 [小宇宙链接]"
- "这个小宇宙节目帮我做成PPT [链接]"
- "深度解读这期播客 [喜马拉雅链接]"
- "把这个B站视频转写传到NotebookLM [bilibili链接]"
- "B站视频帮我生成思维导图 [bilibili链接]"

### X/Twitter 帖子
- "把这条推文传到NotebookLM [x.com链接]"
- "这篇推文线程帮我生成报告 [x.com链接]"
- "深度分析这条推文 [twitter.com链接]"

### YouTube 视频
- **直接传 URL 给 NotebookLM，不下载字幕！** NotebookLM 原生支持 YouTube
- "把这个YouTube视频做成播客 [YouTube URL]"
- "这个视频帮我生成思维导图 [YouTube URL]"

### 本地文件
- "把这个PDF上传到NotebookLM /path/to/file.pdf"
- "这个Markdown文件生成PPT /path/to/file.md"
- "这个EPUB电子书生成播客 /path/to/book.epub"
- "把这个Word文档做成思维导图 /path/to/doc.docx"
- "这个PowerPoint生成Quiz /path/to/slides.pptx"
- "把这个扫描PDF做成报告 /path/to/scan.pdf"（自动OCR）

### 搜索关键词
- "搜索 'AI发展趋势' 并生成报告"
- "搜索关于'量子计算'的资料做成播客"

### 混合使用
- "把这篇文章、这个视频和这个PDF一起上传，生成一份报告"

### 深度分析模式（递归提问）
- "深度分析这本书 /path/to/book.epub"
- "提炼这篇文章的核心观点 [URL]"
- "递归提问分析这个PDF /path/to/file.pdf"
- "帮我深度解读这个视频 [YouTube URL]"

### 深度分析 + 飞书文档
- "深度分析这本书并写入飞书 /path/to/book.epub"
- "分析这篇文章后创建飞书文档 [URL]"
- "递归提问并生成飞书文档 /path/to/file.pdf"

## 自然语言 → NotebookLM 功能映射

| 用户说的话 | 识别意图 | NotebookLM 命令 |
|-----------|---------|----------------|
| "生成播客" / "做成音频" / "转成语音" | audio | `generate audio` |
| "做成PPT" / "生成幻灯片" / "做个演示" | slide-deck | `generate slide-deck` |
| "画个思维导图" / "生成脑图" / "做个导图" | mind-map | `generate mind-map` |
| "生成Quiz" / "出题" / "做个测验" | quiz | `generate quiz` |
| "做个视频" / "生成视频" | video | `generate video` |
| "生成报告" / "写个总结" / "整理成文档" | report | `generate report` |
| "做个信息图" / "可视化" | infographic | `generate infographic` |
| "生成数据表" / "做个表格" | data-table | `generate data-table` |
| "做成闪卡" / "生成记忆卡片" | flashcards | `generate flashcards` |
| "深度分析" / "提炼核心观点" / "递归提问" / "深度解读" | deep-analysis | 自动生成10个问题并递归提问 |
| "写入飞书" / "创建飞书文档" / "生成飞书文档" / "保存到飞书" | feishu | 创建飞书文档并写入内容 |

**如果没有明确指令**，默认只上传不生成任何内容，等待用户后续指令。

## 工作流程

### Step 1: 识别内容源类型

Claude 自动识别输入类型：

| 输入特征 | 识别为 | 处理方式 |
|---------|-------|---------|
| `https://mp.weixin.qq.com/s/` | 微信公众号 | MCP 工具抓取 |
| `https://youtube.com/...` 或 `https://youtu.be/...` | YouTube | 直接传递给 NotebookLM |
| `xiaoyuzhoufm.com` 或 `ximalaya.com` 或 `bilibili.com` | 播客/视频 | Get笔记 API 转写 → TXT |
| `x.com` 或 `twitter.com` | X/Twitter 帖子 | 内置代理级联抓取 → TXT |
| `https://` 或 `http://`（付费网站） | 付费墙网页 | 内置付费墙绕过（UA伪装+archive.today）→ TXT |
| `https://` 或 `http://` | 网页 | 直接传递给 NotebookLM |
| `/path/to/file.pdf` | PDF 文件 | markitdown 转 Markdown → TXT |
| `/path/to/file.epub` | EPUB 电子书 | **Python ebooklib** 提取文本 → TXT（避免 Calibre） |
| `/path/to/file.docx` | Word 文档 | markitdown 转 Markdown → TXT |
| `/path/to/file.pptx` | PowerPoint | markitdown 转 Markdown → TXT |
| `/path/to/file.xlsx` | Excel | markitdown 转 Markdown → TXT |
| `/path/to/file.md` | Markdown | 直接上传 |
| `/path/to/image.jpg` | 图片（OCR） | markitdown OCR → TXT |
| `/path/to/audio.mp3` | 音频 | markitdown 转录 → TXT |
| `/path/to/file.zip` | ZIP 压缩包 | 解压 → markitdown 批量转换 |
| 关键词（无URL，无路径） | 搜索查询 | WebSearch → 汇总 → TXT |

### Step 2: 获取内容

**微信公众号**：
- 使用 MCP 工具 `read_weixin_article`
- 返回：title, author, publish_time, content
- 保存为 TXT：`/tmp/weixin_{title}_{timestamp}.txt`

**播客/视频（小宇宙/喜马拉雅/B站）**：
- 通过 Get笔记 API 获取完整转写文本

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-anything-to-notebooklm](https://github.com/joeseesun/qiaomu-anything-to-notebooklm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
