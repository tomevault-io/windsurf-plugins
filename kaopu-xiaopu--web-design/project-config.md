---
trigger: always_on
description: Web 视觉设计 SKILL。输入 PRD / 参考 URL / 截图 / 关键词（任意组合），先产出一份标准化 DESIGN.md 设计规范，用户确认后据此生成 UI/UX、视觉、动效、响应式全部达标的 web 代码。专攻 web 端：Landing Page、Portfolio、产品页、博客、个人站、SaaS 介绍页等。当用户说"帮我做个网站""设计一个页面""参考 XX 做一个""把这个截图/PRD 做成网页""做一个 landing page""出一份 design 规范"时触发。不用于后端、数据库、纯逻辑 bug 修复。
---


# Web Design

Web 端视觉设计 SKILL。两阶段工作流：**先出规范（DESIGN.md），再出代码。**

DESIGN.md 是显式文件产物，保存到项目目录，可跨项目复用、可手动修改、可给其他工具消费。

## 核心流程

```
Phase A  理解需求（灵活输入）──→  Phase B  输出 DESIGN.md（用户确认）──→  Phase C  生成项目代码
```

**启动时先检测**（按优先级）：
1. 项目里已经有 DESIGN.md 了吗？→ 有则沿用/修改/重建
2. 项目里有 PRD.md 吗？→ 有则进入 A4（PRD 驱动流程）
3. 都没有 → 正常走 Phase A

---

## Phase A: 理解需求

输入是灵活的，以下任意组合都行，不要强制用户走固定路径：

| 输入 | 处理方式 |
|------|---------|
| **参考 URL** | 抓取 HTML+CSS，提取 Token，motion audit |
| **截图 / 设计稿** | 从视觉提取气质、色温、密度、字体风格 |
| **关键词 / 描述** | "暗色克制衬线风格" → 从 style-seeds 匹配 Token |
| **品牌名** | "做成类似 Linear 的" → 先查 `references/design-systems/`，有则直接读取；无则抓取该网站 |
| **混合** | "参考这个 URL 但配色要暖一些" → 提取 + 覆盖 |
| **PRD 文档** | 读取 PRD.md 的产品定位、页面结构、设计交接区 → 分析竞品风格 → 推导设计方向 |

### A1. URL 分析（有参考 URL 时）

**意图判断**：参考（提取设计语言，做独立新页面）vs 复刻（用户明确说"复刻/clone"）。

**抓取流程**（按优先级尝试）：

1. **首选：Playwright 爬虫**（真实浏览器，能绕反爬、渲染 SPA、模拟滚动）
   ```bash
   python3 scripts/crawl_website.py --url [URL] --output ./crawl-output --scroll-delay 600
   ```
   输出：每屏视口截图 + tokens.json + structure.json + styles.css
   
2. **备选：Token 提取脚本**（轻量，适合简单静态站）
   ```bash
   python3 scripts/extract_design_tokens.py --url [URL] --format json
   ```

3. **降级：手动 curl**（脚本都不可用时）
   ```bash
   curl -Ls [URL]
   ```

综合三类输入判断：实际体验（氛围节奏）> 截图（局部气质）> Token 数值（精确参数）。Playwright 爬虫的视口截图可直接作为"体验"和"截图"两类输入。

**Motion Audit**（参考站有滚动联动动效时必做）：
- 触发信号：动效跟随滚动连续变化 / 模块间有 stagger-pin-parallax / 引用 GSAP-Lenis 等
- 记录每个动效区块的 trigger、driver、elements、effect、timing

**降级链路**：
- Playwright 爬虫失败 → 查 `references/design-systems/` 有无该品牌预置文件（58 个品牌）
- 预置文件也无 → `extract_design_tokens.py` 尝试静态提取
- 静态提取也失败 → 手动 curl CSS
- 全部失败 → 要求用户提供截图，从视觉手动建 Token

### A2. 对话引导（无参考物时）

快速锁定风格方向，不要机械走问答清单：

**核心问题**（按需选问，不全问）：
- 亮色 vs 暗色？
- 衬线 vs 无衬线？
- 克制留白 vs 丰富饱满？
- 强调色偏好？
- 有没有喜欢的网站或风格参考？

从 `references/style-seeds.md` 匹配最近似的种子（10 种预设方向），展示给用户确认或混搭。

用户提到具体品牌时，查阅 `references/design-systems/INDEX.md` 按类别检索，按需读取对应品牌文件（58 个真实网站设计规范，每个 ~300 行）。品牌文件只含静态设计系统，动效仍需按交互档位补充。

### A3. 交互档位确认（不可跳过）

| 档位 | 体验感 |
|------|--------|
| **L1** 精致静态 | 优雅 hover + 柔和入场 |
| **L2** 流畅交互 | 滚动 reveal、视差、导航变化 |
| **L3** 沉浸体验 | 滚动驱动时间线、pin、光标跟随、转场 |

L2 追问：reveal 风格（fadeInUp/scaleIn）、视差、导航变化
L3 追问：section pin、光标效果、转场、是否允许 GSAP/ScrollTrigger/Lenis

查阅 `references/interaction-patterns.md` 获取对应档位的完整代码。

### A4. PRD 驱动（检测到项目级规范文件时）

**启动时自动扫描项目根目录**，按优先级：
1. `PRD.md` / `prd.md`
2. `SPEC.md` / `spec.md`
3. `README.md` 或任何 `.md` — 若含「产品定位 / Target Users / Pages / 核心页面」等关键字段

**不绑定特定工具或模板**。只要能从文档里抽到下列字段，就足以作为设计输入：

| 字段（中/英别名） | 用途 |
|-------------------|------|
| 产品名称 / Product / Name | Hero 文案 |
| 一句话定位 / Tagline / Pitch | Hero 副标题、定调 |
| 目标用户 / Target Users / Audience | 色彩温度、字体风格决策 |
| 核心页面 / Pages / Screens | Phase C 生成清单 |
| 竞品 / Competitors / References | Phase A 风格参考起点 |
| 技术栈 / Tech Stack | Phase C 代码生成方案 |
| 设计交接 / Design Notes / 调性 | 直接继承的硬约束 |

**竞品风格分析**：
- PRD 提到竞品 → 优先查 `references/design-systems/` 是否已有预置
- 有预置 → 读取该品牌规范作为起点
- 无预置 → 调用 `scripts/crawl_website.py` 抓取提取
- PRD 没提 → 按产品类型 + 调性从 `references/style-seeds.md` 匹配最近种子

**提取不到关键字段时**：不要瞎猜，回到对话引导（A2）补齐。

**与用户确认**（示例）：
> 我从 `{文件名}` 读到了：
> - 产品：{名称} — {一句话定位}
> - 目标用户：{画像}
> - 参考竞品：{列表 或 "无"}
> - 建议的风格方向：{1-2 个候选，各附一句理由}
>
> 这个方向对吗？

确认后进入 A3（交互档位），然后 Phase B。

---

## Phase B: 输出 DESIGN.md（必走）

**必须生成 DESIGN.md 文件。** 按 `references/design-md-template.md` 模板输出 9 个章节：

1. **Visual Theme & Atmosphere** — 设计哲学、氛围关键词、一句话定调
2. **Color Palette & Roles** — 完整 CSS 变量定义（含 RGB 辅助值）
3. **Typography Rules** — 字体族、Google Fonts URL、字号层级表、禁止字体
4. **Component Stylings** — 按钮/卡片/导航/链接/标签等完整 CSS（含所有状态）
5. **Layout Principles** — 网格、间距梯度、容器宽度
6. **Depth & Elevation** — 阴影体系
7. **Animation & Interaction** — 动效档位、入场/滚动/悬停/特效的完整代码
8. **Do's and Don'ts** — 设计护栏、Anti-Patterns（至少 8 条）
9. **Responsive Behavior** — 断点、触摸目标、折叠策略

**文字装饰规则**：生成 Color Palette 和 Typography 后，按 `references/text-decoration-rules.md` 决策表对标题逐级判断是否添加渐变/投影。

**展示摘要给用户确认**，确认后保存 DESIGN.md 到项目目录，再进入 Phase C。

---

## Phase C: 生成项目代码（必走）

### C1. 确认场景 + 收集内容

问用户做什么页面，收集具体内容。查阅 `references/scene-defaults.md` 获取该场景的布局和组件基线。

**常见场景的内容需求**：

| 场景 | 需要收集 |
|------|---------|
| Landing Page | 标题、副标题、Feature 列表、CTA、Social proof |
| 个人站 / Portfolio | 名字、头衔、项目列表、关于、联系方式、博客列表 |
| Blog | 名称、文章列表/内容、作者信息、分类标签 |
| 产品页 | 产品名、卖点、功能截图、定价、FAQ |

用户信息不全时主动问；用户说"你先做，内容我后面填"则用合理的占位内容。

### C2. 设计框架 + 生成代码

1. **检查项目环境**：框架、路由、样式方案、现有组件和 assets
2. **框架决策**：有项目 → 沿用；无项目 → 最小可行方案
3. **严格按 DESIGN.md 生成**：
   - 所有颜色通过 CSS 变量引用
   - 字体按 DESIGN.md 定义使用
   - 交互按 DESIGN.md 指定档位实现
   - 不违反 Do's and Don'ts
4. **图片策略**：用户素材 > 项目 assets > 参考站 URL 占位 > Unsplash
5. **图标策略**：项目现有库 > lucide-react > 内联 SVG（参考 `references/icon-library.md`）

### C3. 审计

- **DESIGN.md 合规检查**：代码是否严格遵循规范
- **参考物差异审计**（有参考时）：Token / 版式 / 字体 / 交互 / 素材逐项比对
- **响应式验证**：至少移动端 + 桌面端
- **质量清单**：读 `references/quality-checklist.md` 逐项自检

---

## 首页爆点原则（Landing Page 专属）

一个 Web 平台最重要的是**落地页（首页）**。其他页面保持规范、风格统一即可，唯有首页必须满足下面两条硬性原则，不可妥协。

### 原则一：3 个「卧槽」爆点 + 1 个巧思

爆点位置**固定**，每个位置必须有一个让用户"卧槽"的视觉冲击：

| # | 位置 | 要求 | 典型招式 |
|---|------|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KAOPU-XiaoPu/web-design](https://github.com/KAOPU-XiaoPu/web-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
