---
trigger: always_on
description: 面向开发者的 AI 编程工具选型静态网站。基于 Astro 5 SSG + YAML 数据 + 双语（中/英）+ Vercel 部署。
---

# CodePick — AGENTS.md

面向开发者的 AI 编程工具选型静态网站。基于 Astro 5 SSG + YAML 数据 + 双语（中/英）+ Vercel 部署。

## 常用命令

```bash
npm run dev                    # 本地开发服务器
npm run build                  # 构建（每次改动后务必验证）
npm run preview                # 预览构建结果
npm run monitor                # Python 数据监控脚本
npm run dashboard              # Python 数据 dashboard
npm run gen-og                 # 重新生成 OG 图片（需要 sharp）
npm run gen-social-post        # 生成社交媒体发帖模板
npm run gen-juejin             # 生成掘金发帖模板
npm run update-github-tools    # 自动更新 GitHub 开源工具数据
npm run update-closed-source-tools  # 自动更新闭源工具数据
npm run update-all-tools       # 同时运行上面两个脚本
```

## 项目结构

```
data/
  tools/*.yaml        # 工具数据（每个工具一个文件，共 20 个）
  apis/*.yaml         # API 源数据（ark, bailian, ollama, openrouter）
  plans/*.yaml        # 方案数据（工具 + API 组合，共 14 个）

src/
  lib/
    data.ts           # YAML 加载 + 归一化 + TypeScript 接口定义
    i18n.ts           # 双语翻译系统（150+ 翻译 key，t()、localePath() 等）
    faq-data.ts       # FAQ 内容数据

  page-views/         # 页面模板组件（19 个，含所有页面 UI 逻辑）
    HomePage.astro
    ToolDetailPage.astro
    ToolsIndexPage.astro
    ToolsOverviewPage.astro    # 全景图（比分矩阵 + 决策树）
    ToolsFreePage.astro
    ToolsChinaPage.astro
    ToolsTerminalPage.astro
    ToolsVscodePage.astro
    ToolsBuilderPage.astro     # 应用构建型工具（Bolt / v0 / Lovable 等）
    ToolsComparePage.astro     # 工具对比矩阵页
    PlanDetailPage.astro
    CompareArticlePage.astro
    CompareIndexPage.astro
    GuidesArticlePage.astro
    GuidesIndexPage.astro
    FaqPage.astro
    CodingPlanPage.astro       # Coding Plan 专题页
    DealsPage.astro            # 优惠/折扣汇总页
    PromptsPage.astro          # 提示词 / 规则模板页

  pages/
    index.astro              # 根路径语言重定向（JS 检测 navigator.language）
    api/openrouter.json.ts   # OpenRouter API 端点
    zh/                      # 中文路由（薄包装，透传 lang="zh" 给 page-views）
      index.astro / faq.astro / coding-plan.astro / deals.astro / prompts.astro
      rss.xml.ts             # 中文 RSS feed（聚合 compare + guides）
      tool/[id].astro / plan/[id].astro
      compare/index.astro / compare/[slug].astro
      guides/index.astro / guides/[slug].astro
      tools/index|free|china|vscode|terminal|overview|builder|compare.astro
    en/                      # 英文路由（结构与 zh/ 完全镜像，透传 lang="en"）
      rss.xml.ts             # 英文 RSS feed

  content/
    config.ts         # Content Collections 定义（4 个集合）
    compare/*.md      # 对比文章（中文，Astro Content Collections）
    compare-en/*.md   # 对比文章（英文版本）
    guides/*.md       # 使用指南（中文）
    guides-en/*.md    # 使用指南（英文版本）

  layouts/
    BaseLayout.astro  # 主布局：OG/canonical/JSON-LD/hreflang/语言切换/Analytics

  components/
    ToolCard.astro    # 工具卡片（复用于各分类页）
    PlanCard.astro    # 方案卡片（含组合预览）
    ScenarioCard.astro   # 首页场景速查卡片
    QuickPicker.astro    # 30 秒选型问卷（3 题 → 推荐方案）
    ScoreBar.astro       # 评分可视化条形图
    CompareTable.astro   # 功能对比矩阵
    FilterBar.astro      # 工具筛选控件
    TableOfContents.astro  # 文章目录组件
    Comments.astro       # 评论组件
    ShareBar.astro       # 文章/工具/方案底部分享栏（zh: X/微博/掘金/复制；en: X/LinkedIn/Reddit/HN/Copy）

  styles/
    global.css        # 全局样式（TailwindCSS 指令）

scripts/
  check-freshness.mjs       # 检测 YAML 数据新鲜度，超期自动创建 GitHub Issue
  gen-og-image.mjs          # 生成 OG 图片（1200×630，需要 sharp）
  gen-social-post.mjs       # 生成社交发帖模板
  gen-juejin-post.mjs       # 生成掘金发帖模板
  monitor.py                # 检测工具版本/定价变化，创建 GitHub Issue
  update-github-tools.mjs   # 自动更新 GitHub 开源工具版本
  update-closed-source-tools.mjs  # 自动更新闭源工具数据

.github/workflows/
  monitor.yml       # 每周一 09:00（北京）运行 monitor.py
  freshness.yml     # 每周一 09:00（北京）运行 check-freshness.mjs
```

## 双语架构（i18n）

### 路由规则

- 所有页面都有 `/zh/...` 和 `/en/...` 两个版本
- Astro 5 i18n 配置：`defaultLocale: 'zh'`，`prefixDefaultLocale: true`
- 根路径 `/index.astro` 用 JS 检测 `navigator.language`，自动跳转 `/zh/` 或 `/en/`
- `vercel.json` 中配置了 15 条 301 重定向，将旧路径（如 `/tools`）重定向到 `/zh/tools`

### page-views 模式

路由文件（`src/pages/zh/` 和 `src/pages/en/`）只是薄包装：

```astro
---
import ToolDetailPage from '../../../page-views/ToolDetailPage.astro';
import { getToolLocalized } from '../../../lib/data';
// ...
---
<ToolDetailPage tool={tool} lang="zh" />
```

**所有 UI 逻辑都在 `src/page-views/` 中**，路由文件仅负责传入 `lang` 属性和获取数据。

### 翻译系统

使用 `src/lib/i18n.ts` 中的 `t(key, lang)` 函数：

```typescript
import { t, localePath, getLocaleFromUrl } from '../lib/i18n';

// 翻译
t('nav.tools', lang)        // zh: "工具库"，en: "Tools"
t('score.coding', lang)     // zh: "编程"，en: "Coding"

// 带 locale 前缀的路径
localePath('/tools/free', lang)  // "/zh/tools/free" 或 "/en/tools/free"

// 获取当前语言
const lang = getLocaleFromUrl(Astro.url)  // 'zh' 或 'en'
```

### 内容集合（Content Collections）

4 个集合定义在 `src/content/config.ts`：
- `compare` — 中文对比文章
- `guides` — 中文使用指南
- `compare-en` — 英文对比文章
- `guides-en` — 英文使用指南

获取集合时使用 `getCollectionName(base, lang)` 工具函数：

```typescript
import { getCollectionName } from '../lib/i18n';
const collection = getCollectionName('compare', lang);  // 'compare' 或 'compare-en'
```

## 工作流偏好

- **写完改动后不要自动 `git add/commit`**，等用户确认后再提交
- **提交前先运行 `npm run build` 验证构建无报错**

## 新增工具流程

1. 在 `data/tools/` 创建 `<tool-id>.yaml`（参考下方字段规范）
2. 如需新增 API 源，在 `data/apis/` 创建对应 YAML
3. 如需新增方案，在 `data/plans/` 创建对应 YAML
4. 运行 `npm run build` 确认 `/zh/tool/<id>` 和 `/en/tool/<id>` 页面正常生成
5. 等用户确认后提交

## 工具 YAML 完整字段规范

```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WhiteWorld/codepick](https://github.com/WhiteWorld/codepick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
