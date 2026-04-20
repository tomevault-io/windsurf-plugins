---
trigger: always_on
description: > 简洁的 CFA Level 1 学习 vault。用户按教材顺序 (V1→V10) 阅读 PDF，Agent 辅助生成笔记和追踪习题。
---

# CFA L1 Vault — Claude Code 规则

> 简洁的 CFA Level 1 学习 vault。用户按教材顺序 (V1→V10) 阅读 PDF，Agent 辅助生成笔记和追踪习题。

核心循环：**阅读教材 → 宽笔记 → 遇到问题 → 详细笔记 → 做题 → 追踪**

---

## 1. 设计原则

1. **教材为主**: 用户的主要学习时间在阅读 PDF 教材上，Agent 是辅助
2. **按需展开**: 只在用户开始某周学习时生成宽笔记，只在用户提问时生成详细笔记
3. **严格贴合教材**: 宽笔记和详细笔记必须严格跟随教材结构和内容
4. **Obsidian 原生**: 充分利用 Obsidian 原生功能——Bases、Callouts、Properties、Wikilinks、Embeds、Canvas、Mermaid
5. **学习收尾要可审计**: 当用户说今天学完了，应优先基于 Git 变化而不是模糊记忆来判断今天实际学了什么

---

## 2. Vault 结构

```
Home.md              — Dashboard（含 callouts、embeds）
Plan.md              — 21 周计划（带 frontmatter）
Glossary/            — 术语库
  Glossary.base      — 数据库视图（table/cards，按科目分组）
  README.md          — 说明文档
  Library/
    *.md             — 每个术语一个笔记（tag: term）
Formulas/            — 公式库
  Formulas.base      — 数据库视图（table/cards，按科目分组）
  README.md          — 说明文档
  Library/
    *.md             — 每个公式一个笔记（tag: formula）
Practice/            — 习题追踪
  Practice.base      — 数据库视图（错题本/全部/标星/按错误类型）
  README.md          — 说明文档
  V{n}-{Subject}/Week{X}/
    Week-{XX}-LM{nn}-Practice.md  — LM 习题集合 note
    Week-{XX}-LM{nn}-Q{qq}.md     — 每道题一个笔记（tag: question）
V1-Quant/ ... V10-Ethics/  — 各 Volume 笔记
  Week1/ Week2/ ...        — 每周子文件夹
    Week-{XX}.md           — week root
    Week-{XX}-LM{nn}-*.md  — LM 子笔记
    {知识点}.md            — 仅在需要时创建的详细笔记
PDFs/                — 教材 PDF
scripts/             — 工具文件
  agent-log.md       — 学习日志
  session.ctx        — 当前学习状态
```

---

## 3. 快速工作流路由

| 用户说 | 执行 |
|--------|------|
| `今天学什么` / `开始第 X 周` / `本周学什么` | 查 `Plan.md` → 创建 week root + LM 子笔记 → 参考 `.claude/vault-skills/start-week-note/SKILL.md` |
| `开始LM01` / `开始 LM1` | 创建/更新 LM 子笔记 + 同步收集教材 Practice Problems → 参考 `.claude/vault-skills/build-lm-note/SKILL.md` + `collect-lm-practice/SKILL.md` |
| `补全 LM3` / `生成 LM3 笔记` | 创建/更新 LM 子笔记 → 参考 `build-lm-note/SKILL.md` |
| `检查这周有没有漏内容` | 对照 Plan.md + PDF TOC 审计 → 参考 `audit-week-coverage/SKILL.md` |
| 问某个概念 / `展开这个知识点` | 创建详细笔记 → 参考 `expand-topic-note/SKILL.md` |
| `这题不懂` / `这道题我还是不理解` | 先在 question note 写完整解析，必要时补充详细笔记 → 参考 `expand-topic-note/SKILL.md` |
| `润色这篇笔记` | 保持范围，提升质量 → 参考 `polish-cfa-note/SKILL.md` |
| `添加术语` / `提取术语` | 创建 `Glossary/Library/{Term}.md` → 参考 `extract-glossary-terms/SKILL.md` |
| `添加公式` / `提取公式` | 创建 `Formulas/Library/{Formula}.md` → 参考 `extract-formulas/SKILL.md` |
| `润色这个术语` / `润色这个公式` | 润色库条目 → 参考 `polish-memory-note/SKILL.md` |
| `结束今天学习` / `今天学完了` | 基于 git 变化做学习收尾 → 参考 `close-study-session/SKILL.md` |
| `复习一下` / `今天复习什么` | 间隔复习 → 参考 `spaced-review/SKILL.md` |
| `更新进度` / `同步进度` | 扫描 vault 更新 Home/Plan 进度 → 参考 `sync-progress/SKILL.md` |
| `我哪里最弱` / `分析错题` | 分析薄弱点 → 参考 `weak-spot-analyzer/SKILL.md` |
| `开始学习` / `热身` | 推送热身卡 → 参考 `daily-warmup/SKILL.md` |

---

## 4. Skill 调用速查

### Vault Local Skills（`.claude/vault-skills/`）

| 需要做什么 | Skill 路径 |
|-----------|-----------|
| 开始一周学习 | `.claude/vault-skills/start-week-note/SKILL.md` |
| 创建/补全某个 LM 笔记 | `.claude/vault-skills/build-lm-note/SKILL.md` |
| 收集某个 LM 的教材习题 | `.claude/vault-skills/collect-lm-practice/SKILL.md` |
| 审计某周覆盖完整性 | `.claude/vault-skills/audit-week-coverage/SKILL.md` |
| 展开一个知识点为详细笔记 | `.claude/vault-skills/expand-topic-note/SKILL.md` |
| 润色现有笔记 | `.claude/vault-skills/polish-cfa-note/SKILL.md` |
| 从笔记提取术语 | `.claude/vault-skills/extract-glossary-terms/SKILL.md` |
| 从笔记提取核心公式 | `.claude/vault-skills/extract-formulas/SKILL.md` |
| 润色术语/公式库条目 | `.claude/vault-skills/polish-memory-note/SKILL.md` |
| 结束当天学习并总结 | `.claude/vault-skills/close-study-session/SKILL.md` |
| 间隔复习术语/公式/错题 | `.claude/vault-skills/spaced-review/SKILL.md` |
| 同步 Home/Plan 进度 | `.claude/vault-skills/sync-progress/SKILL.md` |
| 分析薄弱点和错题模式 | `.claude/vault-skills/weak-spot-analyzer/SKILL.md` |
| 每日学习前热身 | `.claude/vault-skills/daily-warmup/SKILL.md` |

### Obsidian Skills（`.claude/obsidian-skills/skills/`）

| 需要做什么 | Skill 路径 |
|-----------|-----------|
| 写 `.md` 笔记（任何类型） | `.claude/obsidian-skills/skills/obsidian-markdown/SKILL.md` |
| 修改 `.base` 数据库视图 | `.claude/obsidian-skills/skills/obsidian-bases/SKILL.md` |
| 创建 `.canvas` 知识图谱 | `.claude/obsidian-skills/skills/json-canvas/SKILL.md` |
| Obsidian CLI 批量操作 | `.claude/obsidian-skills/skills/obsidian-cli/SKILL.md` |
| 提取网页内容 | `.claude/obsidian-skills/skills/defuddle/SKILL.md` |

---

## 5. 宽笔记 (Wide Notes)

**触发**: 用户说 "今天学什么"、"开始第 X 周"、"本周学什么" 或 "本周学 V1 LM1-6"

宽笔记分为两层：

1. **Week Root**
   - 文件：`V{n}-{Subject}/Week{X}/Week-{XX}.md`
2. **LM Child Note**
   - 文件：`V{n}-{Subject}/Week{X}/Week-{XX}-LM{nn}-{English Slug}.md`

**Week Root Frontmatter**:
```yaml
---
tags: [week-root, wide-note, V1, quant]
volume: V1
subject: Quant
week: 1
lm_range: "LM1–6"
note_role: week-root
---
```

**LM Child Frontmatter**:
```yaml
---
tags: [wide-note, lm-note, V1, quant]
volume: V1
subject: Quant
week: 1
lm: 1
lm_title: "Rates and Returns"
page_range: "3-44"
parent: "[[Week-01]]"
source_pdf: "PDFs/cfa-program2026L1V1.PDF"
---
```

**Week Root 内容要求**:
- week root 负责范围、顺序、LM 出链、公式索引、问题索引
- 它可以有高层摘要，但不应承担全部 LM 的完整正文覆盖
- 必须链接该 week 的每个 LM 子笔记
- 不要预先生成详细笔记链接列表

**LM Child 内容要求**:
- 严格按教材章节结构，覆盖该 LM 的所有知识点
- 先根据 PDF TOC 生成 `## Coverage Checklist`
- 每个 checklist item 都要在正文中被覆盖，或在 `## End Matter` 中明确承认
- 每个知识点 1-3 句话概括，公式用 LaTeX `$$...$$` 列出
- 使用 Obsidian callouts 标注重要内容：
  - `> [!formula]` — 核心公式

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qiuzan2001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
