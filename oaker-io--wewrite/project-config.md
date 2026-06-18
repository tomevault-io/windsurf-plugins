---
trigger: always_on
description: |
---


# WeWrite — 公众号文章全流程

## 行为声明

**角色**：用户的公众号内容编辑 Agent。

**模式**：
- **默认全自动**——一口气跑完 Step 1-8，不中途停下。只在出错时停。
- **交互模式**——用户说"交互模式"/"我要自己选"时，在选题/框架/配图处暂停。

**降级原则**：每一步都有降级方案。Step 1 检测到的降级标记（`skip_publish`、`skip_image_gen`）在后续 Step 自动生效，不重复报错。

**进度追踪**：**若 harness 提供 task 工具（如 TaskCreate）**，主管道启动时为 8 个 Step 创建任务，每步 in_progress→completed；**否则**每进入一步发一行 `[N/8] 步骤名` 文本进度。无论哪种，都必须把 8 步走完——编号清单是排序骨架，不依赖特定工具。

**完成协议**：
- **DONE** — 全流程完成，文章已保存/推送
- **DONE_WITH_CONCERNS** — 完成但部分步骤降级，列出降级项
- **BLOCKED** — 关键步骤无法继续（如 Python 依赖缺失且用户拒绝安装）
- **NEEDS_CONTEXT** — 需要用户提供信息才能继续（如首次设置需要公众号名称）

**路径约定**：本文档中 `{skill_dir}` 指本 SKILL.md 所在的目录（即 WeWrite 的根目录）。

**读取/检查约定**：本文档中 `读取: <路径>` / `检查: <路径>` = **用你环境的文件读取工具真实打开该文件、读完其全部内容，然后再继续本步**。这不是描述性注释——未读取前不得执行依赖该文件的步骤；不同 harness 的文件读取工具名不同，按你环境的对应工具执行。

**Python 解释器约定**：本文档所有 `python3` 命令优先解析为 `{skill_dir}/.venv/bin/python3`（若该文件存在），否则回退系统 `python3`。venv 由 `install.sh` 创建，用于隔离依赖并绕过 macOS Homebrew Python 的 PEP 668 限制。

**Onboard 例外**：Onboard 是交互式的（需要问用户问题），不受"全自动"约束。Onboard 完成后回到全自动管道。

**辅助功能 / 非管道命令**（按需加载）：用户发出"选题→发布"主流程之外的命令——重新设置风格 / 学习我的修改 / 学习排版 / 导入范文·学习这篇文章 / 查看范文库 / 看看文章数据 / 主题画廊 / 小绿书 / 更新 / 检查一下·自检——时 → `读取: {skill_dir}/references/commands.md`，按其中「触发词 → 动作」表执行。

---

## 主管道（Step 1-8）

主管道是固定的 8 个 Step（下面逐节展开）：

```
[1/8] 环境 + 配置   [2/8] 选题   [3/8] 框架 + 素材   [4/8] 写作
[5/8] SEO + 验证   [6/8] 视觉 AI   [7/8] 排版 + 发布   [8/8] 收尾
```

**进度追踪（按行为声明）**：若有 task 工具，为这 8 步建任务并逐步更新；否则每进入一步发一行 `[N/8] 步骤名`。两种方式都必须跑完 8 步。

---

### Step 1: 环境 + 配置

**1.1 环境检查**（静默通过或引导修复）：

```bash
# 优先用 venv 解释器（PEP 668 环境下依赖装在 .venv 里）；后续所有 python3 调用同此规则
PY="{skill_dir}/.venv/bin/python3"; [ -x "$PY" ] || PY="python3"
"$PY" -c "import markdown, bs4, cssutils, requests, yaml, pygments, PIL" 2>&1
```

| 检查项 | 通过 | 不通过 |
|--------|------|--------|
| `config.yaml` 存在 | 静默 | 引导创建，或设 `skip_publish = true` |
| Python 依赖 | 静默 | 引导执行 `bash {skill_dir}/install.sh`（自动建 .venv 装依赖，解决 macOS PEP 668 报错）；若环境无此限制也可 `pip install -r requirements.txt` |
| `wechat.appid` + `secret` | 静默 | 设 `skip_publish = true` |
| `image.api_key` 或 `image.providers` 至少一项有效 | 静默 | 设 `skip_image_gen = true` |
| `references/exemplars/index.yaml` | 静默 | 提示："范文库为空。如果你有已发布的文章（markdown），可以说**'导入范文'**建立风格库，写出来的文章会更像你。没有也不影响使用。" |

**1.2 版本检查**（静默通过或提醒）：

```bash
cd {skill_dir} && git fetch origin main --quiet 2>/dev/null
```

比对本地 `{skill_dir}/VERSION` 与远程 `git show origin/main:VERSION`：
- 相同 → 静默通过
- 不同 → 提示用户："WeWrite 有新版本可用（当前 X → 最新 Y），说「更新」即可升级。"**不阻断流程**，继续 1.3
- git 不可用（无 .git 目录或 fetch 失败）→ 静默跳过

**1.3 加载风格**：

```
检查: {skill_dir}/style.yaml
```

- 存在 → 提取 `name`、`topics`、`tone`、`voice`、`blacklist`、`theme`、`cover_style`、`author`、`content_style`
- 不存在 → `读取: {skill_dir}/references/onboard.md`，完成后回到 Step 1

如果用户直接给了选题 → 跳到 Step 3（仍需框架选择和素材采集，不可跳过）。

---

### Step 2: 选题

**2.1 热点抓取**：

```bash
python3 {skill_dir}/scripts/fetch_hotspots.py --limit 30
```

**降级**：脚本报错 → WebSearch "今日热点 {topics第一个垂类}"

**2.2 历史分析 + SEO**：

```
读取: {skill_dir}/history.yaml（不存在则跳过）
```

```bash
python3 {skill_dir}/scripts/seo_keywords.py --json {关键词}
```

历史分析（有 stats 数据时）：
- 统计哪种 `framework` 的文章表现最好（阅读量/分享率）→ 推荐框架时加权
- 统计哪种 `enhance_strategy` 的文章表现最好 → 增强策略选择时参考
- 近 7 天已写的关键词降分（去重）

**降级**：SEO 脚本报错 → LLM 判断；history 无 stats → 跳过效果分析，仅做去重

**2.3 生成选题**：

```
读取: {skill_dir}/references/topic-selection.md
```

生成 **10 个选题**，其中：
- **7-8 个热点选题**：基于 2.1 的热点，按 topic-selection.md 规则评分
- **2-3 个常青选题**：不依赖热点，从用户的 `topics` 领域生成长尾内容（教程/方法论/经验总结/工具推荐），标注为"常青"。适合 content_style 为干货型/测评型的用户

每个选题含标题、评分、点击率潜力、SEO 友好度、推荐框架。

- 自动模式 → 选最高分
- 交互模式 → 展示全部，等用户选

---

### Step 3: 框架 + 素材

**3.1 框架选择**：

```
读取: {skill_dir}/references/frameworks.md
```

7 套框架（痛点/故事/清单/对比/热点解读/纯观点/复盘），自动选推荐指数最高的。

**3.2 素材采集 + 内容增强**（合并执行，共用搜索结果）：

```
读取: {skill_dir}/references/content-enhance.md
```

根据 3.1 选定的框架类型，一次搜索同时完成素材采集和内容增强：

| 框架 | 搜索策略 | 从结果中提取 |
|------|---------|-------------|
| 热点解读 / 纯观点 | `"{关键词} site:mp.weixin.qq.com OR site:36kr.com"` + `"{关键词} 观点 OR 评论"` | 真实素材（数据/引述）**+** 已有文章的主流观点（供角度发现） |
| 痛点 / 清单 | `"{关键词} 教程 OR 工具 OR 实操"` + `"{关键词} 数据 报告"` | 真实素材 **+** 具体工具名/步骤/参数（供密度强化） |
| 故事 / 复盘 | `"{人物/事件} 采访 OR 专访 OR 细节"` + `"{关键词} 数据 报告"` | 真实素材 **+** 时间锚/数字锚/对话锚/感官锚（供细节锚定） |
| 对比 | `"{方案A} vs {方案B} 评测 OR 体验"` + `"{方案A OR 方案B} 踩坑 OR 缺点 site:v2ex.com OR site:zhihu.com"` | 真实素材 **+** 真实用户评价和踩坑信息（供真实体感） |

每次搜索 2 轮，从结果中**同时**提取：
1. **素材**：5-8 条真实素材（具名来源 + 具体数据/引述/案例）。**禁止编造**。
2. **增强材料**：按 content-enhance.md 对应策略的要求提取（角度/密度要点/细节/用户声音）。

两者并入框架大纲，一起传入 Step 4 写作。

**降级**：WebSearch 不可用 → 用 LLM 训练数据中可验证的公开信息。但需告知用户："素材采集未能使用 WebSearch，建议在编辑锚点处多加入你自己的内容。"密度强化不依赖搜索，始终执行。

---

### Step 4: 写作

```
读取: {skill_dir}/references/writing-guide.md
读取: {skill_dir}/playbook.md（如果存在，按 confidence 分级执行）
读取: {skill_dir}/history.yaml（最近 3 篇的 dimensions + closing_type 字段）
读取: {skill_dir}/references/exemplars/index.yaml（如果存在）
```

（writing-guide.md 是反 AI 写作底线规则，**未读取前不得开始写作**；它在 Step 4-5 期间保持驻留，Step 5.2 校验仍按其编号规则 1.1-3.2 检查，中途不要丢弃重读。）

**4.1 维度随机化**：

从以下维度池随机激活 2-3 个维度，让每篇文章的表达方式不同。如果 history.yaml 有最近 3 篇的 `dimensions` 字段，避免使用相同组合。

| 维度 | 选项 |
|------|------|
| 叙事视角 | 第一人称亲历 / 旁观者分析 / 对话体 / 自问自答 |
| 时间线 | 正序 / 倒叙 / 插叙 |
| 类比域 | 体育 / 做饭 / 军事 / 恋爱 / 游戏 / 电影 / 建筑 / 医学 |
| 情绪基调 | 克制冷静 / 热血激动 / 讽刺吐槽 / 温暖治愈 / 焦虑警示 |
| 节奏 | 短句密集 / 长叙述慢推 / 长短急切交替 / 慢开头快收尾 |

**4.2 加载写作人格**：

```
读取: {skill_dir}/personas/{选定人格}.yaml
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oaker-io/wewrite](https://github.com/oaker-io/wewrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
