---
trigger: always_on
description: 一句话生成任何领域的知识学习网站。AI自动理解主题、创作内容、生成页面、部署上线。适用于任何需要系统学习的知识领域：进化心理学、大模型术语、化学元素、历史事件等。
---


# Knowledge Site Creator - 通用知识学习网站生成器

**AI理解主题，自动创作内容，生成网站，一键部署。**

## 核心理念

**设计系统优先**：
- 复用设计语言（极简主义、配色、布局、交互模式）
- 不复用具体页面代码
- AI根据主题重新创作所有内容

**通用学习模式**（核心功能）：
- **闪卡（Flashcard）** - 快速记忆
- **学习（Learn）** - 渐进式学习
- **测试（Quiz）** - 知识检验
- **索引（Index）** - 快速查找
- **进度（Progress）** - 学习追踪

**零模板依赖**：
- 不再 `cp -r` 复制模板
- AI参考设计系统，生成新页面
- 所有文案、统计、介绍都由AI创作

## 触发方式

- "生成一个XXX学习网站"
- "创建XXX知识网站"
- "做个XXX学习工坊"

示例：
- "生成一个进化心理学概念学习网站"
- "创建量子力学基础概念网站"
- "做个中医经络穴位学习工坊"

## 工作流程

### 用户视角（一句话）

```
用户："生成一个进化心理学学习网站"

AI自动执行：
✓ 分析"进化心理学"特点和价值
✓ 生成30个核心概念数据
✓ 创作首页文案、统计、介绍
✓ 参考设计系统生成页面
✓ 部署到 Vercel
✓ 返回：https://evolutionary-psychology.vercel.app

完成！
```

---

## 实施流程（AI执行）

### Step 1: 理解主题

AI深入分析主题，输出主题分析：

```javascript
主题分析 {
  领域: "进化心理学",
  特点: "跨学科（生物学+心理学），解释人类行为的底层逻辑",
  价值: "理解人性、改善关系、优化决策",
  受众: "心理学爱好者、自我提升者、教育工作者",
  表达: "科学严谨 + 生活化案例，避免学术术语堆砌"
}
```

**思考问题**：
- 这是什么领域？（学科分类、知识特点）
- 为什么重要？（学习价值、应用场景）
- 目标受众是谁？（背景、需求、痛点）
- 如何表达更好？（语言风格、案例选择）

---

### Step 2: 生成数据 + 网站配置

⚠️ **关键**：生成两个文件，不只是数据！

#### 2.1 生成数据（wordData.js）

**通用数据结构**：
```javascript
const WordRoots = [
  {
    id: 1,
    root: "适应性 (Adaptation)",     // 知识点名称
    origin: "核心理论",               // 分类/来源
    meaning: "通过自然选择进化出的有利特征",  // 一句话解释
    description: "详细说明（200-300字）...",
    examples: [                       // 应用案例/例子（3个）
      {
        word: "恐高症",
        meaning: "对高处的恐惧",
        breakdown: { root: "适应性" },
        explanation: "详细解释..."
      }
    ],
    quiz: {                           // 小测试（4选1）
      question: "以下哪个不是适应性的特征？",
      options: ["选项A", "选项B", "选项C", "选项D"],
      correctAnswer: 2                // 正确答案索引（0-3）
    }
  }
];
```

**生成数量**：默认20-30个，根据主题复杂度调整

#### 2.2 生成配置（siteConfig.js）🆕

**AI创作，完全适配主题**：
```javascript
const siteConfig = {
  // 基础信息
  topic: "进化心理学",
  siteName: "进化心理学概念工坊",
  itemName: "概念",                    // 单个知识点的称呼
  itemCount: 30,

  // 首页Hero区（AI创作）
  hero: {
    title: [
      "30个核心概念",
      "理解人类行为",
      "的底层逻辑"
    ],
    subtitle: "从适应性到配偶选择，系统掌握进化心理学核心框架",
    animation: {
      enabled: true,                   // 是否显示动画
      demoCount: 5                     // 动画展示几个概念
    }
  },

  // 统计卡片（AI生成，匹配主题特点）
  stats: [
    { value: "30", label: "核心概念" },
    { value: "100+", label: "生活应用" },
    { value: "15分钟", label: "每日学习" }
  ],

  // 底部介绍（AI创作）
  footer: {
    tagline: "像理解自己一样理解人性",
    description: "基于进化心理学的科学框架，用30个核心概念解释人类行为背后的生物学逻辑。从配偶选择到亲子关系，从群体合作到情绪反应，让你看懂人性的深层原因。"
  },

  // 按钮文案（AI适配）
  cta: {
    primary: "开始第一个概念 →",
    secondary: "闪卡复习"
  }
};
```

**AI创作原则**：
- `hero.title`: 简洁有力，3行，突出核心价值
- `hero.subtitle`: 具体说明学什么，为什么学
- `stats`: 真实、有说服力的数字，匹配主题特点
- `footer.tagline`: 一句话点题，朗朗上口
- `footer.description`: 2-3句，说清楚是什么、学什么、有什么用

---

### Step 3: 参考设计系统，生成页面

⚠️ **不再复制模板！AI参考设计规范，生成新页面**

#### 3.1 设计系统参考

⚠️ **参考文档**：`references/design-system.md` - 完整的设计规范

**核心要点**：
- **配色**：黄色主题色 (#FBBF24)，灰色系文字和背景
- **字体**：Inter字体族，代码用Courier New
- **风格**：极简主义，大留白，清晰层级
- **组件**：圆角卡片（12px），极浅阴影
- **间距**：8px网格系统，Hero区96px留白

详细配色、字体、间距、组件样式见 `design-system.md`

#### 3.2 生成页面清单

⚠️ **功能参考**：
- `references/core-patterns.md` - 核心学习模式实现
- `references/code-quality.md` - **代码质量标准（必须遵守）**
- `references/seo-best-practices.md` - **SEO优化指南** 🆕
- `references/pwa-setup.md` - PWA配置指南

**代码质量要求**（强制）：
- ✅ **错误处理**：所有 LocalStorage 操作必须有 try-catch
- ✅ **XSS 防护**：使用 textContent/createElement，禁止直接 innerHTML 插入未转义数据
- ✅ **DOM 安全**：所有 DOM 操作前检查元素存在
- ✅ **避免全局污染**：使用模块封装或 IIFE

详细规则见 `references/code-quality.md`。

AI参考设计系统，从零生成以下页面：

1. **index.html** - 首页 🆕
   - Hero区：使用 `siteConfig.hero.title/subtitle`
   - 动画演示：从 `WordRoots` 动态加载前5个（见core-patterns.md §9）
   - 统计卡片：使用 `siteConfig.stats`
   - CTA按钮：使用 `siteConfig.cta`
   - Footer：使用 `siteConfig.footer`

2. **learn.html** - 学习页（见core-patterns.md §5）
   - 渐进式卡片展示
   - 上一个/下一个导航
   - 标记已掌握功能

3. **flashcard.html** - 闪卡页（见core-patterns.md §4）
   - 卡片翻转动画
   - 键盘快捷键（←→翻页，空格翻转）
   - 进度显示

4. **roots.html** - 索引页（见core-patterns.md §7）
   - 标题适配：`${itemName}索引`
   - 搜索框 + 筛选器
   - 卡片网格布局

5. **progress.html** - 进度页（见core-patterns.md §8）
   - 学习统计
   - 已掌握列表
   - 成就系统

6. **root-detail.html** - 详情页
   - 概念详细说明
   - 例子展示
   - 测试题（见core-patterns.md §6）

7. **css/minimal.css** - 样式文件（见design-system.md）
   - 统一设计系统
   - 响应式布局

8. **js/storage.js** - 存储逻辑（见core-patterns.md §3）
   - LocalStorage 进度管理

9. **manifest.json** - PWA 配置（见pwa-setup.md §1）🆕
   - App 名称、图标、主题色
   - 支持安装到主屏幕

10. **sw.js** - Service Worker（见pwa-setup.md §2）🆕
    - 缓存静态资源
    - 支持离线访问

11. **icon-192.png / icon-512.png** - PWA 图标 🆕
    - **自动生成**：使用 PIL 从配置生成（黄色背景 + 主题文字）
    - **不要手动创建**：AI 应自动用 Python PIL 生成

12. **sitemap.xml** - 网站地图（见seo-best-practices.md §4）🆕
    - 列出所有页面URL
    - 提交到搜索引擎

13. **robots.txt** - 爬虫指令（见seo-best-practices.md §5）🆕
    - 允许/禁止抓取规则
    - Sitemap 位置声明

**⚠️ 强制要求：所有 HTML 文件必须包含完整的 meta 标签**

每个 HTML 文件的 `<head>` 必须包含：

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>${siteConfig.siteName}</title>

  <!-- SEO 基础 -->
  <meta name="description" content="${siteConfig.footer.description}">
  <meta name="keywords" content="${siteConfig.topic},学习,知识,${siteConfig.itemName}">
  <meta name="author" content="乔木">
  <meta name="language" content="zh-CN">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-knowledge-site-creator](https://github.com/joeseesun/qiaomu-knowledge-site-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
