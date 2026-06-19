---
trigger: always_on
description: 小红书全能助手 — 文案生成、封面制作、内容发布与管理。当用户要求写小红书笔记、生成小红书文案/标题/封面、发小红书、搜索小红书、评论点赞收藏等任何小红书相关操作时使用。支持一站式从文案创作到自动发布的完整流程。封面AI生图需配置可选环境变量（GEMINI_API_KEY 或 IMG_API_KEY 或 HUNYUAN_SECRET_ID+KEY）。
---


# 📕 小红书全能助手

两大核心能力：**文案创作**（标题+正文+封面图）和 **平台操作**（发布+搜索+互动）。

文案创作默认使用当前对话的主模型，无需额外配置。

---

## 零、查看可用模型（仅当用户询问时）

当用户询问"有哪些模型"、"当前模型"、"可用模型"、"能用什么模型"时，读取配置文件展示：

```bash
# 查看当前主模型
cat ~/.openclaw/openclaw.json | jq -r '.agents.defaults.model.primary // .agents.defaults.model // "未设置"' 2>/dev/null

# 查看所有可用模型（提供商/模型ID - 名称）
cat ~/.openclaw/openclaw.json | jq -r '.models.providers | to_entries[] | .key as $p | .value.models[]? | "\($p)/\(.id) - \(.name)"' 2>/dev/null
```

---

## 一、文案创作流程

当用户要求写笔记、生成文案、创作小红书内容时，按 **标题 → 正文 → 封面图** 三步执行，每步需用户确认后再继续。

### 1.1 生成标题

**优先使用当前对话模型直接生成**，参考 [references/title-guide.md]({baseDir}/references/title-guide.md) 中的规范生成5个不同风格的标题。

核心要求：每个标题使用不同风格，20字以内，含1-2个emoji，禁用平台禁忌词。

**备用方案**：如果用户明确配置了 `XHS_AI_API_KEY` 环境变量并要求使用指定 API，可调用脚本：
```bash
bash {baseDir}/scripts/generate.sh title "内容摘要"
```

**输出后询问用户**：选择哪个标题？可修改或自定义。默认选第一个。

### 1.2 生成正文

**优先使用当前对话模型直接生成**，参考 [references/content-guide.md]({baseDir}/references/content-guide.md) 中的规范，根据选定标题生成正文。

核心要求：600-800字，像朋友聊天的语气，禁用列表/编号，用自然段落呈现，文末5-10个#标签。

**备用方案**：如果用户明确配置了 `XHS_AI_API_KEY` 环境变量并要求使用指定 API，可调用脚本：
```bash
bash {baseDir}/scripts/generate.sh content "完整内容" "选定标题"
```

**输出后询问用户**：是否满意？可要求修改。确认后进入封面图步骤。

### 1.3 生成封面图

封面图结构：1080x1440（3:4），上半部分为主题图片（1080x720），下半部分为纯色底+标题文字（1080x720）。

#### 1.3.1 询问用户选择封面图片来源

**必须先询问用户**：

> 封面图的主题图片，你想怎么来？
> 1. **AI 自动生成** — 根据文案主题自动生成匹配的图片
> 2. **上传自己的图片** — 提供图片路径，我来帮你拼接封面

#### 1.3.2A 用户选择「AI生成」

**继续询问 prompt 方式**：

> AI图片的提示词，你想怎么来？
> 1. **预设推荐** — 我根据你的文案主题自动生成最佳英文prompt
> 2. **自定义提示词** — 你提供想要的画面描述，我来翻译成英文prompt

**预设推荐**：Agent 参考 [references/cover-guide.md]({baseDir}/references/cover-guide.md) 自动生成英文 prompt，展示给用户确认后执行。

**自定义提示词**：用户描述画面，Agent 翻译/优化为英文 prompt，展示确认后执行。

确认 prompt 后，根据主题从 [references/cover-guide.md]({baseDir}/references/cover-guide.md) 配色库选择底色和字色（必须主动搭配，禁止白底黑字）。

##### 生图模型选择策略

**优先尝试当前对话使用的模型**直接生图（如果当前模型支持图片生成）。Agent 在自己的对话环境中直接调用生图能力：
1. 生成 3:2 比例的主题图片，保存到临时文件（如 `/tmp/xhs_ai_img.png`）
2. 然后调用 cover.sh 时传入 `__USER_IMAGE__:/tmp/xhs_ai_img.png`，跳过脚本内置的 API 调用

**如果当前模型不支持生图**（生成失败或明确不具备图片生成能力），**询问用户**：

> 当前模型不支持图片生成，请选择生图方式：
> 1. **Google Gemini** — 需要提供 GEMINI_API_KEY（[获取地址](https://aistudio.google.com/apikey)）
> 2. **OpenAI / OpenAI兼容API** — 需要提供 API Key 和 Base URL
> 3. **其他方式** — 你来提供图片，我帮你拼接封面

用户选择后，设置对应的环境变量再调用 cover.sh：

- **Gemini**：`GEMINI_API_KEY=xxx bash cover.sh "标题" "prompt" ...`
- **OpenAI兼容**：`IMG_API_TYPE=openai IMG_API_KEY=xxx IMG_API_BASE=https://api.openai.com/v1 IMG_MODEL=dall-e-3 bash cover.sh "标题" "prompt" ...`
- **腾讯云混元生图（AIART）**：`IMG_API_TYPE=hunyuan HUNYUAN_SECRET_ID=AKID... HUNYUAN_SECRET_KEY=... HUNYUAN_REGION=ap-guangzhou bash cover.sh "标题" "prompt" ...`
- **其他方式**：用户提供图片路径，走 `__USER_IMAGE__` 模式

若用户之前已提供过 API Key（本次会话中），后续生图直接复用，无需重复询问。

直接调用 cover.sh 的命令格式（仅当需要脚本内置 API 生图时）：

```bash
bash {baseDir}/scripts/cover.sh "标题文字" "英文prompt" [输出路径] [底色hex] [字色hex]
```

#### 1.3.2B 用户选择「上传图片」

用户提供图片路径后，同样搭配底色和字色，执行：

```bash
bash {baseDir}/scripts/cover.sh "标题文字" "__USER_IMAGE__:/path/to/image.jpg" [输出路径] [底色hex] [字色hex]
```

`__USER_IMAGE__:` 前缀会跳过 AI 生成，直接用用户图片裁剪拼接。

#### 封面图前置要求

- ImageMagick（`convert` 或 `magick`）、中文字体（`fonts-noto-cjk`）
- 生图 API Key（仅脚本内置 API 生图时需要，当前模型直接生图则不需要）

### 1.4 文案完成后

询问用户是否要直接发布到小红书。如果要发布，自动进入下方「平台操作」的发布流程。

---

## 二、平台操作

当用户要求发帖、搜索、评论等小红书操作时使用。所有命令在云服务器本地执行，MCP 服务运行在 `http://localhost:18060/mcp`。

### 2.1 前置检查

每次操作前必须先执行：

```bash
bash {baseDir}/check_env.sh
```

返回码：`0` = 正常已登录 → 调用工具；`1` = 未安装 → 安装 MCP 服务；`2` = 未登录 → 扫码登录流程。

### 2.2 调用工具

**⚠️ 极其重要**：小红书 MCP 使用 Streamable HTTP 模式。每次调用都必须：初始化 → 获取 Session ID → 带 Session ID 调用工具。三步在同一个 exec 中执行。

```bash
MCP_URL="${XHS_MCP_URL:-http://localhost:18060/mcp}"

# 初始化并获取 Session ID
SESSION_ID=$(curl -s -D /tmp/xhs_headers -X POST "$MCP_URL" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"openclaw","version":"1.0"}},"id":1}' > /dev/null && grep -i 'Mcp-Session-Id' /tmp/xhs_headers | tr -d '\r' | awk '{print $2}')

# 确认初始化
curl -s -X POST "$MCP_URL" \
  -H "Content-Type: application/json" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","method":"notifications/initialized","params":{}}' > /dev/null

# 调用工具（替换 <工具名> 和 <参数>）
curl -s -X POST "$MCP_URL" \
  -H "Content-Type: application/json" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<工具名>","arguments":{<参数>}},"id":2}'
```

注意：每次调用都必须重新初始化获取新 Session ID，三步必须在同一个 exec 中顺序执行。

### 2.3 可用工具

#### 1. check_login_status — 检查登录状态
- **触发词**: "检查登录"、"登录状态"
- **参数**: 无

#### 2. get_login_qrcode — 获取登录二维码
- **触发词**: "获取二维码"、"扫码登录"
- **参数**: 无
- **返回**: Base64 图片和超时时间

#### 3. delete_cookies — 重置登录状态
- **触发词**: "退出登录"、"重新登录"、"清除登录"
- **参数**: 无
- **注意**: 删除后需要重新扫码登录

#### 4. publish_content — 发布图文内容
- **触发词**: "发小红书"、"发布笔记"、"发图文"
- **参数**:
  - `title`: 标题，≤20字（必填）
  - `content`: 正文，≤1000字（必填）
  - `images`: 图片本地绝对路径数组（必填），如 `["/tmp/food1.jpg"]`

#### 5. publish_with_video — 发布视频内容
- **触发词**: "发视频"、"发布视频笔记"
- **参数**:
  - `title`: 标题（必填）
  - `content`: 描述（必填）
  - `video`: 视频文件本地绝对路径（必填）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [echo-ikun/xhs-autopost-skill](https://github.com/echo-ikun/xhs-autopost-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
