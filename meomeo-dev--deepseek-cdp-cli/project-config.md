---
trigger: always_on
description: Use this skill when you need DeepSeek to research public web information, fact-check claims, collect source links, summarize or compare uploaded files, or analyze images and UI screenshots into structured observations.
---


# DeepSeek

本手册给人和无状态 LLM 提供 `deepseek` CLI 的基本使用框架。

## 核心认知

- `deepseek` 驱动本机 Chrome / Chromium 里的 DeepSeek 网页，不是裸 HTTP 客户端。
- 首次使用先运行 `deepseek auth login`。登录后，普通命令默认使用 DeepSeek 专用 profile。
- `--clone-chrome-profile` 是 legacy：从普通 Chrome source profile 复制 DeepSeek 登录态；不要把它当成 auth profile。
- `deepseek` 适合开放网页搜索、摘要、候选链接整理、文件上传、会话继续和导出。
- `deepseek` 不负责结构化行情数据库、数值型截面查询或研究状态管理。
- 首选命令是 `deepseek`。`deepseek-cdp` 和 `deepseek-cdp-cli` 只是兼容别名。
- `deepseek version` 会返回当前包版本、GitHub 地址和 License，适合 LLM 做能力/版本自检。
- `deepseek skillbook` 会原样返回当前 `SKILL.md`。
- 当前只适配 macOS（only supported platform）。

## 安装

```sh
npm install -g deepseek-cdp-cli
```

常用入口：

```sh
deepseek --help
deepseek version
deepseek skillbook
```

## 使用节奏

### 1. 登录

```sh
deepseek auth login
```

`auth login` 会打开可见 Chrome。用户在 DeepSeek 网页完成登录后，CLI 会保存专用 profile：

```text
~/.deepseek-cdp-cli/auth/chrome-profile
```

清理专用 profile：

```sh
deepseek auth logout
```

`auth logout` 只删除 `~/.deepseek-cdp-cli/auth`，不会删除用户自己的 Chrome profile。

### 2. 看运行计划

```sh
deepseek plan
```

`plan` 用来确认本次命令的 request family、是否会 auto-isolate、是否会 attach 到已有 CDP 浏览器。
`plan` 不接收 `--chat-mode`、`--file`、`--message` 这类发送参数；
这些参数只放在 `deepseek reply`。

默认规则：

- 有 auth profile：普通 one-shot 命令优先使用专用 profile，留在 managed 路径。
- 没有 auth profile：普通 one-shot 命令默认尝试 attach 到 `http://127.0.0.1:9222`。
- 显式 `--browser-id`、`--browser-mode`、`--cdp-url` 会固定意图；冲突时 fail-closed。
- auth profile 或 legacy clone 请求不会 auto attach 到用户现成浏览器。

### 3. 发第一条消息

```sh
deepseek reply \
  --message "用三句话介绍这个项目" \
  --headless \
  --quiet \
  --format text
```

`--quiet` 只影响 runtime logs，不会移除文本输出里的 `sessionId`。

```text
sessionId: ...
```

拿到 `sessionId` 后继续：

```sh
deepseek reply --session-id <sessionId> --message "继续" --quiet --format text
```

默认 `reply` 使用 Expert + DeepThink，Search 当前关闭：

```text
--chat-mode expert --deep-think on --search off
```

DeepSeek 官网当前因算力不足暂时隐藏 Expert 的联网搜索和附件入口。
因此 `--chat-mode expert --search on` 与
`--chat-mode expert --file ...` 都会提前 fail-closed。需要联网时先用
Instant：

```sh
deepseek reply \
  --message "联网检索一个公开事实并给出来源" \
  --headless \
  --chat-mode instant \
  --deep-think on \
  --search on \
  --quiet \
  --format text
```

### 4. 找回网页里的旧会话

如果网页里有会话，但本地还没有，先同步 catalog：

```sh
deepseek sync-session --headless
deepseek list-sessions
```

边界：

- `deepseek list-sessions` 只列本地已经持久化的会话，不是远端账号历史目录。
- 不带 `--session-id` / `--session-file` 的 `deepseek sync-session` 只同步网页可观测到的 online catalog，并把结果落成本地 catalog snapshot / placeholder。
- 这一步不会逐个进入会话页。
- 这一步不会同步 transcript / branches / messages。
- 这一步不是稳定 public API。

### 5. 刷新单个会话正文

如果网页 transcript 比本地新，先 targeted sync：

```sh
deepseek sync-session --session-id <sessionId> --headless
```

`deepseek sync-session --session-id <id>` 或 `--session-file <path>` 才是显式单会话 transcript sync。

记住：

- `reply` 不会自动 sync。
- `export-session` 不会自动 sync。
- `list-branches` 不会自动 sync。

### 6. 继续、查看分支、导出

```sh
deepseek reply --session-id <sessionId> --message "继续说" --stream --quiet --format text
deepseek list-branches --session-id <sessionId>
deepseek export-session --session-id <sessionId> --format text --output ./session.txt
deepseek export-session --session-id <sessionId> --format markdown --output ./session.md
deepseek export-session --session-id <sessionId> --format json --output ./session.json
```

## 投研与网页搜索

投研默认命令：

先创建 prompt 文件。命名规范：

```text
deepseek-research-<topic-kebab>-<YYYYMMDD>.prompt.md
```

示例：

```sh
cp deepseek-research-template.prompt.md deepseek-research-feilong-inquiry-20260425.prompt.md
```

再执行：

```sh
deepseek reply \
  --message "$(cat deepseek-research-feilong-inquiry-20260425.prompt.md)" \
  --headless \
  --chat-mode instant \
  --search on \
  --deep-think on \
  --quiet \
  --format text
```

`deepseek-research-template.prompt.md` 模板。原样复制；只修改 `已知上下文` 和 `任务`，其他部分保持原样。

```markdown
深度搜索, 逐步的, 每步搜索一个查询, 逐步串行地从互联网中搜索收集所需的多个维度信息:
1. ...
2. ...
3. ...
4. ...
5. ...
6. ...
7. ...

我们需要同时探索多个方向, 针对研究中的缺口进行补充搜索。

当上下文未提供必要属性所需的数据信息时：
- 若该信息**无法通过上下文推断或合理派生**，则使用显式标记默认占位填充（templated placeholder, UPPER_CASE）提示用户补充
- 若该信息**可通过上下文中的其他信息推断得出**，则可直接使用推断值
- **禁止**自行生成"看似真实"但实际未提供的具体数据（如虚构的ID、时间戳、用户名等）

附加格式要求:

1. 用摘要或关键词概括那个连续段落的核心内容，并放在问题之前, 总-分的写作方法。
2. 在连续段落的开头或结尾加入信号词, 信号词标记方式与回复格式保持一致, 比如markdown时请使用 `**` 加粗标记，并保持格式统一。信号词中英文对照, 术语英文放到括号里。只需要使用标记方式标记信号词即可, 无需出现“信号词”字眼，保持自然行文。
3. 自然地集中出现该段落内容主题独有的专有名词或术语，但不要过度堆砌，保持自然行文。
4. 使用任何能别出“章节边界”、并促使章节内部术语集中的标记方式分割长文档(默认为(markdown+mathjax+补充HTML片段(内联样式))作为回复格式, 否则按照用户指定格式回复), 使用明显的标题或分隔符标记出章节边界，使得每个章节内部的块更加同质。
5. 对于上下文要额外注意, 由于注意力机制, 召回上下文时会召回紧密衔接的内容，上下文可能无法清晰切割, 因此需要注意用户问题的严格边界。
6. 每次回复需要加上唯一ID(格式: 回复ID: <最最重要的那个关键词,驼峰写法>-<YYYYMMDD日期>-<\d{3}自增长>), 方便后序引用(引用格式: `[inner: 回复ID]`)。

数学公式使用 mathjax 表达, 特殊token表达到markdown代码块中。

已知上下文:

- 附件文件名 , 内容一句话说明
- 若有多个则列出...

任务:

你的检索问题
```

适合：

- 公开网页搜索
- 跨站点线索发现
- 公司官网、交易所公告、权威媒体、行业媒体、公开研报摘要补充阅读
- 候选链接清单整理

不适合：

- 结构化行情数据库
- 数值型截面和时间序列主查询
- 最终投资结论

证据边界：

- `deepseek` 输出默认是开放网页候选证据，不直接等于高可信结论。
- 聚合摘要、二手转载、自媒体、论坛内容、未标明原始出处的网页整理都应降权。
- 重要事实尽量回到原始公告、公司原文、交易所或巨潮页面、权威媒体原文、正式研报页面。

常用问题写法：

- “请联网检索并概括截至 YYYY-MM-DD 最近一周最关键的 4 条催化和 4 条风险，优先公司公告、权威媒体、券商研报摘要。”
- “请列出最值得二次核验的 5 个公开网页链接，并标明来源类型，不要直接给投资建议。”

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meomeo-dev/deepseek-cdp-cli](https://github.com/meomeo-dev/deepseek-cdp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
