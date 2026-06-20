---
trigger: always_on
description: |
---


# 大师 · 行业蒸馏术

> 「同事.skill 蒸馏一个人，女娲.skill 蒸馏一种思维方式，大师.skill 蒸馏一整个行业的 OS。」

## 核心理念

一个好的行业 master skill **不是行业百科**，是**可运行的行业认知操作系统**：

- 这行的人用什么**心智模型**看问题？（镜片）
- 这行的标准**playbook** 是什么？（决策启发式）
- 这行的人怎么**沟通**？（行业表达 DNA）
- 什么算**好工作**？（质量基准）
- 外行才会犯什么错？（反模式）
- 这行的工具栈和工作流今年怎么变的？（时效层）

**关键区分**：捕捉的是 HOW the field thinks，不是 WHAT happens to be in the news this week。

---

## 范围约束（重要）

- **粒度**：一个 master skill 覆盖**一个细分领域**（例：「LLM agent 基础设施」「跨境电商运营」「足踝外科」），不要试图覆盖整个泛行业（「软件」「医疗」会让 OS 稀薄到没用）
- **目标用户**：行业内 / 想入行 / 服务这行的人。不是给完全外行做扫盲
- **不替代实操工具**：master skill 是「思维顾问」，不是 RPA。不要在里面塞「自动操作 X 后台」之类
- **版权安全**：不存全本书 / 完整字幕 / 长段原文。只保留结构化摘要 + 来源元数据 + 极短引用

---

## 路径占位符 (iter 24 重要)

prompts 里两个不同的目录占位符:

| 占位符 | 含义 | 例子 |
|--------|------|------|
| `{master_skill_dir}` | master-skill 仓库 (本仓库) 的 `skill/` 目录, 工具脚本住在这里 | `~/master-skill/skill/` 或 `/Users/foo/master-skill/skill/` |
| `{skill_dir}` | 当前正在生成 / 更新 的**行业** skill 目录, 调研笔记 / SKILL.md / sub-skills 都写到这里 | `~/.claude/skills/llm-agent-infra-master/` |

工具脚本 (source_verifier.py / cold_detector.py / quality_check.py / collectors / ingest / transcribe) **不复制**到生成的行业 skill 目录, agent 直接调 `{master_skill_dir}/tools/...`. 行业 skill 只装载 prompts + research notes + SKILL.md + sub-skills + cli/.

如果运行环境的 master-skill 路径变了, agent 替换占位符即可; 不需要改任何工具源代码.

---

## 执行流程

### Phase 0: 入口分流

收到用户输入后判断路径：

| 用户输入 | 路径 | 示例 |
|---------|------|------|
| 明确的细分行业 | **直接路径** → Phase 0A | 「我做跨境电商，给我做一个 master skill」 |
| 模糊的需求 | **诊断路径** → Phase 0B | 「我想转 AI 行业，但不知道哪个细分」 |
| 已存在的 skill | **更新路径** → Phase 0C | 「update 大师 LLM-agent-infra」 |

---

### Phase 0A: 行业澄清（直接路径）

确认 6 个维度（用户没主动提的字段就问，主动给的就不要重问）：

1. **行业 / 细分领域**（必填）— 例：`LLM agent infra`，不是 `AI`；`跨境电商运营`，不是 `电商`
2. **聚焦方向**（可选）— 全景画像 vs 某个角度优先（例：技术视角 vs 商业视角）
3. **地域 / 语言区**（可选）— global / 中文圈 / 美国 / 欧洲。影响信源选择
4. **用户身份**（可选）— 从业者 / 学习者 / 投资人 / 咨询。影响 master skill 的语气与决策视角
5. **本地素材**（强烈推荐询问）— 「你手上有这行的一手素材吗？行业报告 PDF / 内部文档 / 你写过的文章 / 长访谈 transcript / 视频字幕？有的话直接丢给我，比网上搜到的二手转述质量高得多」
6. **新建 or 更新**（可选）— 检查 `~/.claude/skills/{slug}-master/` 是否已存在

用户没主动给 → 默认全景 + global + 从业者视角 + 无本地素材，确认后推进。
用户提供了本地素材 → 标记为**本地语料模式**，Phase 1 采集策略相应调整。

确认后 → Phase 0.5。

---

### Phase 0B: 需求诊断（模糊路径）

用户不确定要蒸哪个细分时，反推合适的范围。

#### Step 1: 锚定核心目标

最多 1-2 轮追问，定位用户真正的需求维度：

| 维度 | 典型表达 | 反推方向 |
|------|---------|---------|
| 谋生 / 转行 | 「想入行 X」「做这个能赚钱吗」 | 商业生态视角的 master skill |
| 提效 | 「这行的工作流我搞不清」「文档堆成山」 | 工作流视角的 master skill |
| 决策 | 「不知道该选 A 还是 B」 | 工具选型 + 标准 playbook |
| 学习 | 「想系统学这行」 | 知识正典视角 |
| 创业 | 「想做 X 方向的产品」 | 行业 OS + 信息源（找未被满足的需求） |

#### Step 2: 反推候选

基于用户表达，提议 2-3 个具体的细分领域。每个候选要回答：

- 这个细分**目前真有可调研的公开材料吗**？（冷门领域的 master skill 质量必差）
- 它跟用户的需求多对齐？
- 它跟其他候选的差异在哪？

用户选定后 → Phase 0A 补全 6 个维度 → Phase 0.5。

---

### Phase 0C: 更新路径

读取已存在的 `{slug}-master/SKILL.md`，从 frontmatter 找到 `last_research_date`。基于距今多久决定刷新强度：

| 距今 | 刷新策略 |
|------|---------|
| < 1 月 | 拒绝刷新，告知用户最近刚更新过 |
| 1-3 月 | 仅 Track B（工具）+ Track C（工作流）+ Track E 中的「最新动态」节 |
| 3-6 月 | 加上 Track A（看 top figures 是否有大动作）+ Track F（标准是否有新版） |
| > 6 月 | 全部 6 轨重跑，但保留旧版 archive |

不要每次都重写整个 skill — 行业的**核心 OS** 通常 1-2 年才动一次，**工具 / 工作流**才是高频变量。

---

### Phase 0.5: 创建 Skill 目录

在调研开始之前完成：

```
{HOST_SKILLS_ROOT}/{slug}-master/
├── SKILL.md                          # 最终产物
├── meta.json                         # 元数据 (industry, locale, profile, last_research_date, source_count)
├── scripts/                          # 工具脚本（继承自 master-skill 仓库 + 行业特化）
└── references/
    ├── research/                     # 6 轨调研结果（必存）
    │   ├── 01-figures.md             # 行业大佬
    │   ├── 02-tools.md               # 工具地图
    │   ├── 03-workflows.md           # 工作流 / SOP
    │   ├── 04-canon.md               # 知识正典（书 / 论文 / 课）
    │   ├── 05-sources.md             # 信息源（newsletter / podcast / 会议 / 社区）
    │   └── 06-glossary.md            # 术语 + 标准 + 法规
    ├── synthesis.md                  # Phase 2 提炼结果（行业 OS）
    └── sources/                      # 一手素材（用户提供 + 网络下载的字幕等）
        ├── books/
        ├── transcripts/
        └── articles/
```

**HOST_SKILLS_ROOT 解析**：
- Claude Code: `~/.claude/skills`
- OpenClaw: `~/.openclaw/skills`
- Codex: `~/.codex/skills`
- Hermes: `~/.hermes/skills`
- 其他 / 不确定：在当前 cwd 下创建 `./{slug}-master/`，最后让用户决定装到哪

**目录创建后立即检查**：
- [ ] 目录已创建
- [ ] 如果 locale 是中文圈：信源策略切换为 B 站原始视频 / 小宇宙播客 / 36氪 / 极客公园 / 晚点 / 财新优先（**永远排除知乎、微信公众号、百度百科**）
- [ ] 如果 locale 是英文圈：默认 Twitter/X / YouTube / Substack / arXiv / 行业 podcast 优先
- [ ] 如果是更新模式：已读取现有 `SKILL.md` 和 `references/research/*.md`，标注哪些信息要刷
- [ ] 如果用户提供了本地素材：复制到 `sources/` 对应子目录，标记为**本地语料模式**

**关键规则（开源分发的核心）**：
- 所有调研文件必须存在 skill 目录内部 `references/research/`，**绝不存到外部目录**
- skill 必须自包含：clone 整个目录就能用
- 每个 subagent 必须把调研写入对应文件。不存文件的调研等于没做

---

### Phase 1: 多源信息采集（六轨 Agent Swarm，wave 结构）

#### 模式判断

| 模式 | 触发 | 策略 |
|------|------|------|
| **纯网络搜索**（默认） | 用户没给本地素材 | 6 个 Agent 全部走网络 |
| **本地语料优先** | 用户提供了 PDF / 行业报告 / transcript | 先按 6 轨分类本地素材，识别缺口，定向网络补充 |
| **纯本地语料** | 用户明确说「只用我给的」 | 只分析本地，不联网 |

#### Wave 结构（重要 — 不是纯并行）

iter 4 figures 跑通后发现：6 轨之间互相是 bootstrap 输入。Track 01 (figures) 的「最大几家公司创始人」依赖 Track 02 (tools) 找到的「关键工具」；Track 02 的「业内人吐槽过哪些工具」反过来依赖 Track 01 找到的人。强行纯并行 → 各轨都凭空猜起点。

执行结构：

**Wave 1（并行）— 「广撒网」轨**：
- Track 04 (canon) — 必读书 / 论文 / 课，搜索路径最独立（看 Goodreads / arXiv / 大学课程页就行）
- Track 05 (sources) — newsletter / podcast / 会议，搜索路径独立（搜「{industry} podcast」「{industry} newsletter」）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swaylq/master-skill](https://github.com/swaylq/master-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
