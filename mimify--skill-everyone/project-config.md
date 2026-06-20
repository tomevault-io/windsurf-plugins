---
trigger: always_on
description: |
---


# Skill Everyone · 万物皆可角色

> 任何你能叫出名字的角色，都可以在这里活起来。

---

## 退出 / 取消

用户在任意阶段说「取消」「算了」「退出」「cancel」「stop」→ 立即停止当前流程，回复：

```
已取消。已生成的文件保留在 $SKILL_DIR/characters/<slug>/（如有），可随时用 /summon update <slug> 继续。
如果什么都没生成，直接忽略即可。
```

之后恢复正常对话模式，不再执行 summon 的任何步骤。

---

## 语言规则

根据用户第一条消息的语言，全程使用同一语言。用中文触发就全程中文，用英文触发就全程英文。

---

## 路径约定

- **执行任何操作前**，先用 Bash 动态解析路径（不依赖任何硬编码目录名或 skill 命令名）：
  ```bash
  # 路径解析优先级：
  # 1. 标准安装位置 ~/.claude/skills/skill-everyone/
  # 2. 如果不存在，再搜索其他位置
  
  _STANDARD_PATH="$HOME/.claude/skills/skill-everyone/.skill-everyone-root"
  
  if [ -f "$_STANDARD_PATH" ]; then
    _MARKER="$_STANDARD_PATH"
  else
    # 回退：搜索其他可能的安装位置
    _MARKER=$(find ~ -maxdepth 6 -name ".skill-everyone-root" 2>/dev/null | head -1)
  fi
  
  if [ -z "$_MARKER" ]; then
    echo "错误：找不到 .skill-everyone-root 标记文件，路径解析失败。"
    echo "请确认 skill-everyone 已正确安装到 ~/.claude/skills/skill-everyone/"
    exit 1
  fi
  SKILL_DIR=$(dirname "$_MARKER")

  # 上级目录 = 框架的 skills 根目录（Codex/Claude Code/其他都适用）
  SKILLS_BASE=$(dirname "$SKILL_DIR")

  # 验证目录实际存在
  if [ ! -d "$SKILL_DIR/prompts" ]; then
    echo "警告：SKILL_DIR=$SKILL_DIR 但 prompts/ 子目录不存在，请检查安装是否完整。"
  fi

  echo "SKILL_DIR=$SKILL_DIR"
  echo "SKILLS_BASE=$SKILLS_BASE"
  ```
  解析后：
  - `$SKILL_DIR` = 本 skill 所在目录（数据和 prompts 都在这里）
  - `$SKILLS_BASE` = 生成的角色 skill 输出到这里的子目录，如 `$SKILLS_BASE/<slug>/SKILL.md`
- **数据目录**（无论先生成哪种模式，始终在这里）：
  `$SKILL_DIR/characters/<slug>/`
  包含：`persona.md`、`world.md`、`meta.json`、`references/`
- **沉浸模式 SKILL.md**：`$SKILLS_BASE/<slug>/SKILL.md`
- **视角模式 SKILL.md**：`$SKILLS_BASE/<slug>-perspective/SKILL.md`
- 两个 SKILL.md 都引用同一个数据目录，数据不随 Skill 输出目录重复存储
- slug 规则：小写字母 + 数字 + 连字符，例如 `geralt-witcher3`、`hermione-novel`、`cloud-ff7`

---

## 何时激活哪个流程

收到用户输入后，先判断路径：

| 输入形式 | 路径 |
|---------|------|
| `/summon <角色名>` 或 `帮我生成XX` 或 `创建XX` | → **主流程：生成新角色** |
| `/summon list` 或 `列出角色` | → **列出已有角色** |
| `/summon add <slug>` 或 `给XX追加材料` | → **追加材料流程** |
| `/summon update <slug>` 或 `更新XX` | → **增量更新流程** |

---

## 工具检查（首次使用时执行）

**触发时机**：用户首次触发 `/summon` 时，在进入 Phase 0 前检查一次。

```bash
# 检查调研工具
YTDLP_OK=false; SCRAPLING_OK=false; JQ_OK=false

command -v yt-dlp &>/dev/null || [ -f ~/.local/bin/yt-dlp ] && YTDLP_OK=true
python3 -c "import scrapling" &>/dev/null && SCRAPLING_OK=true
command -v jq &>/dev/null && JQ_OK=true

echo "yt-dlp=$YTDLP_OK scrapling=$SCRAPLING_OK jq=$JQ_OK"
```

**只提示缺失的工具**（已安装的不提），展示后**直接继续**，不等待用户：

```
─── 调研工具检查 ─────────────────────────────────────

[缺 yt-dlp 时显示]
⚠ yt-dlp 未安装 — 无法提取 B站/YouTube 视频字幕
  安装命令：curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp \
           -o ~/.local/bin/yt-dlp && chmod +x ~/.local/bin/yt-dlp

[缺 scrapling 时显示]
⚠ scrapling 未安装 — 无法绕过 Fandom 等网站的反爬
  安装命令：pip install scrapling[all]

[缺 jq 时显示]
⚠ jq 未安装 — JSON 处理能力受限
  安装命令：sudo apt install jq  或  brew install jq

─────────────────────────────────────────────────────
缺少工具会降低调研质量，但不阻止继续。
```

**如果全部工具已安装**：静默通过，不展示任何提示。

---

## 主流程：生成新角色

### Phase 0：信息采集

读取 `$SKILL_DIR/prompts/intake.md`，执行信息采集。

**最多 3 轮问答**，收集：
1. 角色名 + 所属作品（如果用户没说清楚）
2. 版本确认（如有多版本）
3. 想要的模式（沉浸 / 视角 / 两个都要）
4. 材料来源选择

模式和材料来源**一起展示**，末尾带回复格式示例（读取 intake.md 中的模板，严格按格式输出，不省略任何选项）：

```
想要什么形式的 Skill？

  [A] 沉浸对话   [B] 思维视角   [C] 两个都要

材料怎么来？

  [1] 自动调研   [2] 我来提供文字   [3] 我来上传图片
  [4] 先自动调研再手工补充   [5] 我来定义原创人设

回复两个选项即可，例如：「A, 2」= 沉浸对话 + 我来提供文字
```

**重复检测**：生成 slug 后，检查是否已有同名角色（详见 intake.md "重复角色检测"节）：
- 检查 `$SKILL_DIR/characters/<slug>/meta.json` 和 `$SKILLS_BASE/<slug>/SKILL.md`
- 如有重复，提示用户选择：重新生成 / 追加材料 / 更新设定 / 取消
- 用户确认后才继续

用户选完后**立即**创建目录，不再等待：

```bash
# 数据目录（无论先生成哪种模式都先建好）
# SKILL_DIR 已在路径约定阶段解析，此处直接使用
mkdir -p $SKILL_DIR/characters/<slug>/references/auto
mkdir -p $SKILL_DIR/characters/<slug>/references/auto/source
mkdir -p $SKILL_DIR/characters/<slug>/references/manual/text
mkdir -p $SKILL_DIR/characters/<slug>/references/manual/images
mkdir -p $SKILL_DIR/characters/<slug>/references/manual/source

# 仅选 [5] 原创人设时才建 original 目录（其他路径不建）
# mkdir -p $SKILL_DIR/characters/<slug>/references/manual/original
# mkdir -p $SKILL_DIR/characters/<slug>/references/manual/original/source

# 按用户选择的模式建 skill 目录
# 沉浸模式：mkdir -p $SKILLS_BASE/<slug>
# 视角模式：mkdir -p $SKILLS_BASE/<slug>-perspective
# 两个都要：两个都建
```

---

### Phase 1：材料收集

根据用户选择的路径执行。

#### 路径 1 / 路径 4 的自动调研

读取 `$SKILL_DIR/prompts/research_auto.md`。

**静默执行工具扫描**（不展示、不等待、不阻断）：
- 扫描已安装的辅助 skill（gemini-video, agent-reach, pdf 等）
- 检查本机工具（yt-dlp, scrapling, jq, ffmpeg）
- 根据结果自动调整调研策略

**告知用户开始调研**，然后启动并行调研：

```
正在调研 [角色名]（[作品名]）...
三路并行搜索中，大约需要 2-5 分钟。
```

- **Agent A（基础档案）**：wiki / fandom 主页面，角色简介、背景、关系
- **Agent B（台词与行为）**：台词数据库、剧情摘要、具体场景描述
- **Agent C（社区解读）**：玩家/读者/观众的分析文章、人物解读、争议讨论

**每个 Agent 完成后立即展示单行进度**（不等全部完成）：
```
✓ 档案 — 找到 X 条基础信息
✓ 台词 — 找到 X 条台词/场景（Agent B 完成后展示）
✓ 解读 — 找到 X 篇分析（Agent C 完成后展示）
```

每个 Agent 结果写入：
- `$SKILL_DIR/characters/<slug>/references/auto/wiki.md`
- `$SKILL_DIR/characters/<slug>/references/auto/quotes.md`
- `$SKILL_DIR/characters/<slug>/references/auto/analysis.md`

三路全部完成后，展示汇总质量摘要：

```
─── 调研完成 ────────────────────────────
角色档案    ✓  找到 X 条基础信息
台词/行为   ✓  找到 X 条台词/场景记录
社区解读    ✓  找到 X 篇分析
─────────────────────────────────────────
总计：约 X 条有效信息
```

如果是路径 4，询问：「是否需要补充材料？可以粘贴文字或上传图片（输入"不用了"继续）」

#### 路径 2：手工文字

读取 `$SKILL_DIR/prompts/research_manual_text.md`，引导用户粘贴材料。

- 支持多段输入，用户输入「完成」结束
- 每段让用户说明来源（「第三章」「wiki 引用」「自己总结」等）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MIMIFY/skill_everyone](https://github.com/MIMIFY/skill_everyone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
