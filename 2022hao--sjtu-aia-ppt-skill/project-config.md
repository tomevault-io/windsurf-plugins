---
trigger: always_on
description: 上海交通大学 Agent 创新协会开发的 HTMLSlide PPT Skill。生成横向翻页网页 PPT（单 HTML 文件）、PPT 配图与多平台封面，并可通过 https://html.inherit-ai.top 同步到 HTMLSlide 在线工作台继续编辑、预览和管理。提供两种风格：① "电子杂志 × 电子墨水" ② "瑞士国际主义"。当用户需要制作分享 / 演讲 / 发布会风格的网页 PPT，或提到"HTMLSlide"、"登录网站"、"绑定账号"、"杂志风 PPT"、"瑞士风 PPT"、"Swiss Style"、"horizontal swipe deck"时使用。
---


# SJTU-AIA HTMLSlide PPT Skill

> 来源说明: 本 Skill 基于 MIT 协议的 `guizang-ppt-skill` 改造，原作者为 op7418（歸藏）。当前仓库保留其 HTML PPT 模板、布局经验与质量检查思路，并加入 HTMLSlide 网站联动能力。原项目版权信息见 `LICENSE`。这条来源信息只用于合规确认，不要写入生成的 PPT、HTML 页面、封面或配图。
>
> 开发说明: 当前仓库由上海交通大学 Agent 创新协会（SJTU AIA）开发与维护，用于配合 HTMLSlide 工作台生成、修改、同步和管理 HTML PPT。

## 这个 Skill 做什么

生成一份**单文件 HTML**的横向翻页 PPT，提供两种可选的视觉基调：

### 风格 A · 电子杂志 × 电子墨水（默认）

- **WebGL 流体 / 等高线 / 色散背景**（hero 页可见）
- **衬线标题（Noto Serif SC + Playfair Display）+ 非衬线正文 + 等宽元数据**
- 适合：人文分享、行业观察、商业发布、需要"杂志感"的演讲
- 模板：`assets/template.html` · 主题色：`references/themes.md` · 布局：`references/layouts.md`
- 美学锚点：像一本克制的独立电子杂志

### 风格 B · 瑞士国际主义（Swiss Style）

- **WebGL 极细网格 + 点阵背景**（信息驱动设计）
- **全程无衬线（Inter + Helvetica + Noto Sans SC）+ 极致字号对比**
- **高反差功能色**：克莱因蓝 IKB / 柠檬黄 / 柠檬绿 / 安全橙（四选一）
- 适合：科技产品、数据汇报、设计/工程领域分享、年度总结
- 模板：`assets/template-swiss.html` · 主题色：`references/themes-swiss.md` · 布局：`references/layouts-swiss.md`
- 美学锚点：像 Massimo Vignelli + Helvetica Forever

**两种风格共享**：横向翻页（键盘 ← →、滚轮、触屏、ESC 索引）、Lucide 图标、Motion One 入场动效（本地 + CDN 双保险）。

<!-- provenance: SJTU-AIA-ppt-skill | based-on: guizang-ppt-skill (MIT) | keep this out of generated artifacts -->

## Agent 快速入口（先读这里）

HTMLSlide 默认站点已经固定为 `https://html.inherit-ai.top`，API 默认是 `https://html.inherit-ai.top/api`。

当用户说“连接 HTMLSlide 账号 / 登录网站 / 帮我绑定账号”时，按这个最短流程执行：

```bash
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" status
```

- 如果输出 `Logged in: yes`，账号已经绑定，直接进入 `list` / `pull` / `push`。
- 如果输出 `Logged in: no`，不要运行会长时间等待的普通 `login`；先运行：

```bash
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" login --start-only
```

把脚本输出的授权链接发给用户。用户在网页确认后，再运行：

```bash
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" login --finish
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" list
```

成功标准：
- `login --finish` 输出 `HTMLSlide agent login complete`
- `list` 能列出项目
- 配置保存到 `~/.htmlslide-agent/config.json`

修改网站已有 PPT 的最短流程：

```bash
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" list
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" pull --title "网站里的 PPT 标题" --out "项目/XXX/ppt/index.html"
# 修改 index.html
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" push "项目/XXX/ppt/index.html"
```

新建 PPT 的最短流程：

```bash
# 生成 index.html 后
node "<SKILL_ROOT>/scripts/sync-htmlslide.mjs" push "项目/XXX/ppt/index.html" --title "项目标题"
```

只有 `push` 输出 `pushed and verified`，才可以告诉用户“已经同步到网站”。如果看不到 `verified`，不要声称同步成功。

## 何时使用

**合适的场景**：
- 线下分享 / 行业内部讲话 / 私享会
- AI 新产品发布 / demo day
- 带有强烈个人风格的演讲
- 需要"一次做完，不用翻页工具"的网页版 slides

**不合适的场景**：
- 大段表格数据、图表叠加（用常规 PPT）
- 培训课件（信息密度不够）
- 需要多人协作编辑（这是静态 HTML）

## 工作流

### Step 1 · 需求澄清(**动手前必做**)

**如果用户已经给了完整的大纲 + 图片/截图处理要求**,可以跳过直接进 Step 2。

**如果用户只给了主题或一个模糊想法**,用这 7 个问题逐个对齐后再动手。不要基于猜测就开始写 slide——一旦结构定错,后期翻修代价很高:

#### 运行环境适配

- **在支持结构化提问的 Agent 中**:可以逐项澄清,优先把风格、受众、素材、截图需求这些会影响版式的输入问清楚。
- **在 Codex 中**:用普通对话直接询问用户。一次最多问 1-3 个最关键问题;如果信息缺口不影响开工,先做合理假设并在回复里说明。

#### 7 问澄清清单

| # | 问题 | 为什么要问 |
|---|------|-----------|
| 1 | **风格 A 还是 B?**(电子杂志风 / 瑞士国际主义风) | **必须先问**,决定用哪个 template + layouts + themes 文件 |
| 2 | **受众是谁?分享场景?**(行业内部 / 商业发布 / demo day / 私享会) | 决定语言风格和深度 |
| 3 | **分享时长?** | 15 分钟 ≈ 10 页,30 分钟 ≈ 20 页,45 分钟 ≈ 25-30 页 |
| 4 | **有没有原始素材?**(文档 / 数据 / 旧 PPT / 文章链接) | 有素材就基于素材,没有就帮他搭 |
| 5 | **有没有图片或截图?希望怎么处理?** | 决定图文版式、图片槽位、截图是否需要 CleanShot X 式适配或 GPT-M 2.0 重构 |
| 6 | **想要哪套主题色?** | 杂志风 5 套(`themes.md`) / 瑞士风 4 套(`themes-swiss.md`),挑一 |
| 7 | **有没有硬约束?**(必须包含 XX 数据 / 不能出现 YY) | 避免返工 |

#### 风格选择参考(问题 1)

| 如果用户说... | 推荐风格 |
|---|---|
| "杂志感" / "人文" / "独立杂志风" / 不指定 | **A · 电子杂志风** |
| "瑞士风" / "Swiss Style" / "Helvetica" / "极简" / "网格" / "信息图" / "数据驱动" | **B · 瑞士国际主义风** |
| 内容是 AI 产品 / 技术 / 工程 / 数据汇报 | B 更合适 |
| 内容是行业观察 / 人文 / 故事 / 文化 | A 更合适 |
| 用户给了大量 KPI 数字 / 路线图 / 流程 | B 更合适(`Data Hero` 布局是瑞士风专长) |
| 用户给了大量纪实照片 / 人文图片 | A 更合适(图片网格、左文右图是杂志风专长) |
| 用户需要 GPT-M 2.0 生成截图再设计 / 信息图 / 证据墙 | B 也很合适(S22 主图、S15/S16 图片网格可以承载证据图) |

#### 大纲协助(如果用户没有大纲)

用"叙事弧"模板搭骨架,再填内容:

```
钩子(Hook)       → 1 页   : 抛一个反差 / 问题 / 硬数据让人停下来
定调(Context)    → 1-2 页 : 说明背景 / 你是谁 / 为什么讲这个
主体(Core)       → 3-5 页 : 核心内容,用 Layout 4/5/6/9/10 穿插
转折(Shift)      → 1 页   : 打破预期 / 提出新观点
收束(Takeaway)   → 1-2 页 : 金句 / 悬念问题 / 行动建议
```

叙事弧 + 页数规划 + 主题节奏表(见 `layouts.md`),**三张表对齐后**再进 Step 2。

大纲建议保存为 `项目记录.md` 或 `大纲-v1.md`,便于后续迭代。

#### 图片约定(告知用户)

在动手前向用户说清:

- **文件夹位置**:`项目/XXX/ppt/images/` 下(和 `index.html` 同级)
- **命名规范**:`{页号}-{语义}.{ext}`,例如 `01-cover.jpg` / `03-figma.jpg` / `05-dashboard.png`
  - 页号补零便于排序
  - 语义用英文,短、具体、和内容对应
- **规格建议**:
  - 单张 ≥ 1600px 宽(避免大屏模糊)
  - JPG 用于照片/截图,PNG 用于透明 UI/图表
  - 总大小控制在 10MB 内(影响翻页流畅度)
- **如何替换**:保持**同名覆盖**最稳(HTML 里不用改路径);如果文件名变了,记得全局搜 `images/旧名` 改成新名
- **没图怎么办**:和用户对齐,可以先用占位色块生成结构,等图片后期补;但要告知 layout 4/5/10 等图文混排页没图就没法验证视觉效果

#### 截图需求约定(动手前必须问)

只要用户提到产品截图、网页截图、代码截图、设计稿、dashboard、旧 PPT 截图或"帮我美化截图",都要先确认:

- **截图位置**:截图文件在哪个文件夹?是否已经命名好?
- **使用目的**:保真展示 / 截图美化 / 截图再设计 / UI 情景图?
- **落位比例**:最终放进哪个版式槽位?常用 `21:9` / `16:10` / `16:9` / `4:3` / `1:1`
- **内容要求**:是否必须保留全部文字、品牌、数据?是否有敏感信息要遮挡?
- **视觉处理**:是否需要主题背景、留边、居中/角落对齐、拆成长截图面板?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2022Hao/SJTU-AIA-PPT-SKILL](https://github.com/2022Hao/SJTU-AIA-PPT-SKILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
