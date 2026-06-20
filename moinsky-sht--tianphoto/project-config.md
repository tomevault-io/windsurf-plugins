---
trigger: always_on
description: |
---


# Tianphoto — OpenClaw 优先的智能图文生成工作室

将文章内容转化为**精美的、可编辑的自包含 HTML 网页**，可直接在浏览器中阅读、编辑文字、插入图片，并按需导出 JPG / PNG（适合公众号上传或精修存档）。它不只适用于 Claude Code、Codex、Trae 等 AI IDE / agent 环境，也特别适配小龙虾 OpenClaw；当 OpenClaw 安装了飞书官方 `feishu-openclaw-plugin` 之后，Tianphoto 可以更自然地承接飞书在线文档、通知和周报内容，并一句话生成长图 HTML 再回传到当前会话。

## 核心理念

**网页优先，图片可选。** 先输出一个漂亮的、有设计感的 HTML 页面。用户可以：
- 在浏览器中直接查看效果
- 点击文字即可编辑
- 拖拽/粘贴图片
- 文本粘贴自动净化，回车自动生成干净段落
- 点击底部"保存"按钮（或 Cmd+S）保存修改后的网页
- 点击"导出"按钮生成 JPG / PNG（支持安全切片）

**宿主适配上，Tianphoto 是通用的，但不是平均分配的。** 它可以运行在多种 AI IDE / agent 环境中；不过如果你在 OpenClaw 里使用，尤其是已经安装飞书官方 `feishu-openclaw-plugin` 时，它的优势会更明显: 可以读取飞书在线文档等内容源，也可以直接用一句话把飞书里的材料做成长图 HTML，并把结果继续带回当前会话传播、修改和复用。

## 30 秒快速开始

约定：下文中的 `$SKILL_DIR` 表示**当前正在使用的 `tianphoto` 技能根目录**。不要把路径硬编码成 `~/.claude` 或其他固定目录，始终以当前加载的 skill 目录为准。

1. 用户直接给文章文字或 URL 时，先判断内容模式，再判断 UI 模式，再识别内容模板，再选风格家族和预设，再决定 SVG composition（位置 / 强度 / quietness），最后生成 `<article>` 片段。
2. 把片段保存为临时 HTML 后，调用 `node $SKILL_DIR/scripts/render-image.js ...` 输出桌面网页。
3. 如果用户要品牌横幅，把 logo 放到 `$SKILL_DIR/logos/brand-logo.png`，再用 `/tp logo title 名字` 和 `/tp logo subtitle 副标题` 写入本地配置。
4. 如果用户输入 `/tp ui rule`、`/tp ui free 2` 或 `/tp doctor`，优先处理这些快速指令，不要开始生成。
5. 如果用户只是输入 `/tp style list` 或 `/tp help`，直接返回纯文本速查表，不要开始生成。

## /tp 指令系统

用户可以用以下指令来配置 Tianphoto 的行为：

### 高优先级快速指令

这些指令一旦出现，就应当**优先处理并立即返回结果**，不要继续进入内容分析和页面生成：

### `/tp ui rule`
切换到**规则模式**。后续生成继续使用当前这套强结构、强组件、强校验的稳定排版流程。

### `/tp ui free`
切换到**自由模式**。后续生成默认一次输出 2 个不同方向的抽卡版本。

### `/tp ui free <count>`
切换到自由模式，并设置一次生成的抽卡版本数。默认 `2`，最大 `5`。例如：
- `/tp ui free 2`
- `/tp ui free 4`

### `/tp doctor`
快速检查当前 skill 状态：版本、UI 模式、logo 配置、Chrome 可用性、预设数量、OpenClaw 能力和页面设计质量。
如果附带本地 HTML 文件路径，还会检查：
- 是否用了 `tp-free-*` helper
- 是否存在明显的硬编码主题色
- 是否有危险的 3 列以上网格
- 当前页面属于哪个 `content-template`
- 章节图形是否超量、编号是否断档
- 图形是否放错位置、是否抢标题、是否穿越正文主航道
- 阅读型页面是否误用了 `wx-image-drop-zone`
- metric card 是否过长，不适合移动端扫描
- free 模式是否把大 SVG 放进正文中段，或出现多个无关 SVG 容器

### `/tp logo on`
启用 Logo 功能。提示用户将 Logo 图片放到以下位置：
```
$SKILL_DIR/logos/brand-logo.png
```
文件必须命名为 `brand-logo.png`（或 `.svg` / `.jpg`）。放好后再次运行即可自动嵌入品牌横幅。

### `/tp logo off`
关闭 Logo 功能，生成的页面不显示品牌横幅。

### `/tp logo title <text>`
设置品牌横幅主标题，并持久化到 `$SKILL_DIR/local-settings.json`。

### `/tp logo subtitle <text>`
设置品牌横幅副标题，并持久化到 `$SKILL_DIR/local-settings.json`。

### `/tp style auto`
自动模式（默认）。根据文章内容主题自动匹配最佳预设风格。

### `/tp style <preset-id>`
手动指定预设风格，例如：
- `/tp style nebula-frost` — 星云雾面（科技/AI）
- `/tp style dawn-journal` — 曦白札记（知识/观点）
- `/tp style comet-neon` — 彗星霓光（暗色发布）
- `/tp style jade-zen` — 青玉留白（禅意阅读）

完整的 37 套预设见下方速查表。

### `/tp style list`
列出所有可用预设，附带预览说明。直接返回纯文本速查表，不要开始生成。

### `/tp help`
显示所有可用指令说明。直接返回简短帮助，不要开始生成。

### `/tp version`
显示当前安装的 Tianphoto 版本号，并检查 GitHub 是否有新版本可用。

### `/tp update`
从 GitHub 拉取最新版本，自动升级本地 skill 文件。

### `/tp select auto`
自动内容模式（默认）。AI 根据文章类型自动判断：适合浓缩的内容会精炼提取，适合完整展示的内容会保留全文。

### `/tp select full`
完整保留模式。保留原文所有内容，不做任何删减。适合学术论文、教程手册、完整报告等需要保持内容完整性的文章。输出会较长，像一篇正式排版的完整文章。

### `/tp select compact`
紧凑压缩模式。最大程度精炼内容，提取核心要点。适合做海报预览、快速总结、社交媒体分享卡片。

## 指令处理逻辑

当用户输入 `/tp` 指令时，按以下规则处理：

1. **先看是不是高优先级快速指令**。如果用户消息以 `/tp ui` 或 `/tp doctor` 开头，优先执行，不要继续做内容分析。
2. **`/tp ui rule`** → 执行 `node $SKILL_DIR/scripts/tp-config.js ui rule`，确认后续生成将回到稳定组件化模式
3. **`/tp ui free`** → 执行 `node $SKILL_DIR/scripts/tp-config.js ui free`，确认后续默认一次生成 2 个自由抽卡版本
4. **`/tp ui free <count>`** → 执行 `node $SKILL_DIR/scripts/tp-config.js ui free <count>`，其中 `<count>` 超过 5 时按 5 处理；确认当前自由模式抽卡数
5. **`/tp doctor`** → 执行 `node $SKILL_DIR/scripts/tp-doctor.js`，直接返回诊断结果；如果用户同时给了本地 HTML 文件路径，可以执行 `node $SKILL_DIR/scripts/tp-doctor.js <path>`
6. **`/tp logo on`** → 执行 `node $SKILL_DIR/scripts/tp-config.js logo on`，然后检查 `$SKILL_DIR/logos/` 目录下是否有 `brand-logo.*` 文件（支持 png/jpg/svg）。有则确认已启用；没有则提示用户放置文件到该目录，文件名固定为 `brand-logo`
7. **`/tp logo off`** → 执行 `node $SKILL_DIR/scripts/tp-config.js logo off`，告诉用户后续生成将不包含品牌横幅
8. **`/tp logo title <text>`** → 执行 `node $SKILL_DIR/scripts/tp-config.js logo title "<text>"`，确认新的品牌标题已保存
9. **`/tp logo subtitle <text>`** → 执行 `node $SKILL_DIR/scripts/tp-config.js logo subtitle "<text>"`，确认新的品牌副标题已保存
10. **`/tp style auto`** → 确认已切换为自动匹配模式
11. **`/tp style <id>`** → 验证 id 是否在 presets.json 中存在。存在则确认；不存在则给出最接近的建议
12. **`/tp style list`** → 直接输出预设速查表，不要生成页面
13. **`/tp help`** → 直接输出指令帮助，不要生成页面
14. **`/tp version`** → 读取 `$SKILL_DIR/version.json` 中的 `version` 字段，显示当前版本。然后用以下命令检查远程最新版本：
   ```bash
   curl -s https://raw.githubusercontent.com/Moinsky-sht/tianphoto/main/version.json
   ```
   对比版本号，告诉用户是否需要更新。如果 curl 失败（网络问题），提示用户可以手动访问 GitHub 检查。
15. **`/tp update`** → 执行以下命令升级：
   ```bash
   cd $SKILL_DIR && git pull origin main
   ```
   如果本地有未提交的修改，先提示用户，不要覆盖本地改动。更新完成后读取新的 version.json 确认版本号。
16. **`/tp select auto`** → 确认已切换为自动判断模式（默认）
17. **`/tp select full`** → 确认已切换为完整保留模式
18. **`/tp select compact`** → 确认已切换为紧凑压缩模式

## 生成工作流

### Step 0: 静默检查更新

每次生成前，先在后台检查是否有新版本可用：

```bash
curl -s --connect-timeout 3 https://raw.githubusercontent.com/Moinsky-sht/tianphoto/main/version.json
```

读取本地 `$SKILL_DIR/version.json` 的 `version` 字段，与远程返回的 `version` 对比：
- **版本相同或 curl 失败** → 不提示任何内容，静默继续
- **远程版本更新** → 在回复开头简短提醒一句：「Tianphoto 有新版本 vX.X.X 可用，运行 `/tp update` 升级」，然后继续正常生成

注意：这个检查不应该阻塞生成流程。如果网络慢或失败，直接跳过。

### Step 1: 获取内容

- **纯文本**：直接使用
- **URL**：`node $SKILL_DIR/scripts/fetch-content.js <url>`

### Step 1.5: 确定内容模式（auto / full / compact）

如果用户通过 `/tp select` 指定了模式则直接使用。否则默认 `auto`，按以下规则自动判断：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Moinsky-sht/tianphoto](https://github.com/Moinsky-sht/tianphoto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
