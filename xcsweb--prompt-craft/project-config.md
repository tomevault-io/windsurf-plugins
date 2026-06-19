---
trigger: always_on
description: Prompt Craft v4 — 三层 Prompt 架构 + 3 引擎动画库 + 6 大设计方向。从用户一句需求 → 到设计方向 → 到技术栈 → 到可运行代码，一步到位。
---


# 🚀 Prompt Craft — 网页生成器 v3.0

> **一句话定位**：把"我想做一个官网"这句话，自动翻译为「设计方向 × 技术档位 × 交互签名 × 可运行代码」的完整产出。
>
> **本文件的阅读者**：人类用户（看前 3 节，1 分钟掌握怎么用）+ AI（读完整份，按第 6 节的流程执行）。

---

## 📌 一句话怎么用（给人类用户的 3 句话）

你只需要按下面的句式说话，剩下的全部交给 Prompt Craft：

| 你想要的东西 | 你可以这么说 | 默认档位 |
|-------------|--------------|---------|
| 一个公司/品牌官网，要有设计感 | `做一个 [你的行业] 公司官网` | Editorial 杂志风 + 🚀 快速原型档 |
| 一个炫酷的沉浸式官网，像 Awwwards 那样 | `做一个 [你的行业] 官网，要很酷，像 Awwwards 那样` | Cyberpunk 沉浸风 + 🚀 快速原型档 |
| 一个 3D 产品展示页，像小米 SU7 那样 | `做一个 [产品] 的 3D 展示页面，可以旋转查看` | 3D Product Showcase + Three.js |
| 一个数据可视化大屏，展示 KPI / 实时数据 | `做一个 [主题] 的数据可视化大屏` | 深色科技风 + 🚀 快速原型档 |
| 一个后台管理系统，有表格 / 表单 / 权限 | `做一个 [主题] 的后台管理系统` | 浅色 + Ant Design / Element Plus |
| 一个 API / 后端服务 | `做一个 [业务] 的 REST / GraphQL API` | Node.js + Express + Prisma |
| 我已有一段代码出问题 | `这段代码报错了：[贴错误信息]` | Debug 流程 |

**如果你什么都没指定，我们会用这个默认组合**（你可以随时要求替换）：

- 设计方向：**A · Editorial 杂志博物馆风**（奶油纸色 + Serif 大标题 + 巨量留白）
- 技术档位：**🚀 快速原型档**（单文件 HTML + ES Modules CDN，双击打开即运行）
- 交互签名：从 20 个签名库里默认挑 5 个最常用的 — **遮罩文字入场 / 自定义光标+磁吸按钮 / hover 反色卡片 / Shader Orb / 滚动下划线生长**

想改动？可以说：`改成 Cyberpunk 风格`、`升级为 Vue 3 生产级档`、`用横向滚动代替垂直滚动`、`强调色换成 #2D7CFF`。

---

## 🏛 系统架构（三层 Prompt）

```
                                 用户一句话需求
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
        T1 · 项目规则           T2 · 专家角色           T3 · 任务模板
 （这是"法律法规"，必遵守）  （这是"设计师/工程师"人设）  （这是"施工蓝图"，按步执行）
                              ┌───────────────────────┐
                              │  组合后的完整 Prompt    │
                              │  = T1 + T2 + T3 + 用户需求 │
                              └───────────┬─────────────┘
                                          ▼
                                   可运行代码输出
```

### 每一层解决的问题

| 层级 | 文件来源 | 解决的问题 | 本页自动注入的默认值 |
|------|---------|-----------|-------------------|
| **T1 项目级规则** | `project-rules/*.md` | 技术栈硬约束 — 用什么框架、什么字体、颜色数量、性能预算、哪些第三方库被禁止。 | `artistic-stack.md`（官网类请求）/ 根据需求类型切换 |
| **T2 专家角色** | `expert-personas/*.md` | AI 的"思维人格"。官网类走 *Creative Technologist*（克制、叙事感、滚动故事），常规前端走 *Senior Frontend Engineer*，后端走 *Senior Backend Engineer*。 | 官网类 → Creative Technologist |
| **T3 任务模板** | `task-templates/*.md` | 每类任务的 **CO-STAR** 框架：Context、Objective、Style、Tone、Audience、Response Format、Constraints。这是"AI 的施工步骤"。 | 官网类 → `build-artistic-website.md` |

**执行原则（AI 必须遵守）**：永远不要让用户去看某个 md 文件再回来。所有规则的"精华"，你（AI）要在这里当场消化完，直接输出代码。

---

## 🎨 网页生成器 · 设计 DNA 库

> 详细清单请见 `references/artistic-website-dna.md`（约 280 行）。下面是给用户看的"选单" + 给 AI 看的"速查表"。

### 6 个设计方向（选 1 个作为页面主 DNA）

| 方向 | 色彩（一组默认值） | 字体（标题 + 正文） | 布局关键词 | 典型交互 | 适合行业 |
|-----|-------------------|-------------------|----------|---------|---------|
| **A · Editorial 杂志博物馆风** | 奶油纸 `#F5F3EF` + 墨黑 `#1A1A1A` + 赭石 `#E0512B`（3 色原则） | **Display** Newsreader / Playfair（Serif，巨号，行高 ≥ 1.1）<br>**Body** Inter，栏宽 50-65 字符 | 不对称双栏、巨量留白 14-20vh、section 间用巨型数字分隔 | 遮罩文字入场、纸张噪点、hover 反色卡片、滚动下划线生长、长文编辑滚动 | 设计工作室 / 文化机构 / 高端品牌 / 创意咨询 / 金融科技 / 出版社 |
| **B · Cyberpunk 黑暗沉浸式** | 深黑 `#0A0A0F` + 近白文本 + 洋红 `#FF2E97` + 青 `#00F0FF`（双霓虹） | **Display** Condensed / 窄体几何 sans，全大写宽字距<br>**Body** 小号 sans，数字用等宽字体 | 画布优先（WebGL 占满屏），HTML UI 以 HUD 形式浮在 canvas 上 | Shader Orb、全屏 Canvas 2D、粒子场、滚动驱动、HUD 状态面板、实时时钟 ticker | 游戏工作室 / 互动媒体 / 音乐厂牌 / 科幻 IP / 科技产品发布页 / AI 研究实验室 |
| **C · Playful 游戏探索风** | 蓝天绿草橙砖 + 单个签名色（如法拉利红）作交互元素 | **Display** 圆润几何 sans<br>数字用等宽<br>所有标签文字"钉"在 3D 世界标牌上而非 HTML | 没有传统布局，页面是一张 3D 世界地图，区域间用物理导航 | 3D 世界地图、碰撞 / 物理、实体 Tilt Card、横向滚动章节、用户"驾驶"到达内容 | 个人作品集（技术背景）/ 游戏公司 / 儿童教育 / 创意工具发布 |
| **D · 3D Product Showcase** | 深黑 / 纯白背景 + 产品本身材质色 + 1 个 UI 强调色（电蓝 / 霓虹橙） | **Display** 极细 Sans（Inter 300）+ 等宽字体用于规格数字 | 画布优先——WebGL Canvas 占满屏，HTML UI 以 HUD 浮层叠加 | 360° 产品旋转、颜色/材质切换、热点标注、环境光切换、规格表 | 汽车 / 消费电子 / 家具 / 珠宝 / 工业设备 / 高端产品展示 |
| **E · Neo-Swiss 新瑞士粗野主义** | 瑞士米灰 `#E8E6DF` 背景 + 纯墨 `#0A0A0A` + 瑞士红 `#E11D48` + 瑞士蓝 `#1E40AF` | **Display** Space Grotesk（几何无衬线 500-900）<br>**Body** Inter + JetBrains Mono（数字/代码）| 12 列可见网格线、大字号 border 作为结构、news ticker 横向滚动、编号系统 | 反色 manifesto、项目编号列表 (001/002 →)、实时时钟 ticker、hover 时整行位移 | 设计工作室 / 建筑事务所 / 法律金融 / 出版社 / 学术机构 / 严肃科技品牌 |
| **F · Dark Luxury 深色奢华风** | 深墨 `#0A0A0C` + 象牙白 `#E8E6E1` + 暖金 `#C9A962` + 青铜 `#8B7355` | **Display** Inter 200-300 极细（超大型）+ 金色渐变强调<br>**Body** Inter 300<br>**Specs** JetBrains Mono | 产品占 60-80% 屏幕空间，文字信息以 HUD 形式浮动；黄金分割；大量"暗色空间" | 产品 3D 倾斜展示、金色渐变文字、规格表、青铜色次强调、canvas 2D 金色粒子 | 高端手表 / 珠宝 / 香水 / 奢侈品零售 / 私人银行 / 限量版科技产品 / 高端汽车 |

### 15 个交互签名（"菜单" — 每次选 3-5 个）

> 编号 IS-01 到 IS-15，完整技术细节（参数、ease、参考代码骨架）在 `references/artistic-website-dna.md`。
> 一页内不要超过 **5 个**签名；否则页面变成"游乐场"，用户无法聚焦。

| 编号 | 签名名称（英文） | 一句话效果 | 适用场景 | 推荐方向 |
|-----|----------------|----------|---------|--------|
| IS-01 | Mask Reveal | 文字被 clip-path 遮罩，从下至上"打开"出现 | Hero 大标题、section 标题 | Editorial / Cyberpunk |
| IS-02 | SplitText Stagger | 标题拆成单字符，逐字入场（stagger 0.02-0.05s） | Hero、manifesto 宣言 | Editorial |
| IS-03 | Scroll-driven Line Grow | section 标题下划线随滚动从 0 "生长"到目标宽度 | 所有 section 的小标题标签 | 所有方向通用 |
| IS-04 | Custom Cursor + Magnetic Button | 隐藏原生光标，用圆环跟随；hover 按钮时圆环变文字框并"吸附" | 全站光标系统 + 所有 CTA | 所有方向通用 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xcsweb/prompt-craft](https://github.com/xcsweb/prompt-craft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
