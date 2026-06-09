---
trigger: always_on
description: - 这个项目是一个以本地维护为优先的 Linux.do 社区 skill 索引。
---

# Version 0.1.0

## Project Memory

- 这个项目是一个以本地维护为优先的 Linux.do 社区 skill 索引。
- 当前项目路径：`/Users/xlzhao/Documents/Github 项目咨询/linuxdo-awesome-skills/`。
- Git 仓库根目录已经迁移到 `linuxdo-awesome-skills/` 内部；后续所有 `git status`、提交和推送都必须在该目录执行，不要在父目录 `/Users/xlzhao/Documents/Github 项目咨询/` 操作。
- `data/skills/*.json` 是事实来源，README 和静态页面是展示层。
- GitHub Pages 自定义域名：`https://linuxdo-awesome-skills.tencents.ltd/`；README 的网页入口使用“在线导航页面”作为链接名称。
- README 面向最终读者显示，只保留项目定位、网页入口、技能清单、分类导航和阅读说明。
- 与本项目储备、编写过程、数据结构、发布计划、收录方法相关的内容放在 `CLAUDE.md` 或 `docs/collection-policy.md`，不要放进 README。
- README 表格列优先服务筛选和跳转；避免同时放“用途”和“适合场景”这类语义重复的长列。
- README 和静态页可展示 `前置要求`，用于帮助读者快速判断是否需要特定模型、API、本地脚本、后端服务或素材条件；保持短语化，不写安装教程。
- README 表格下方的技能简介可附 `description`，来源为原项目 `SKILL.md` frontmatter 的 description；中文 description 优先使用原文，英文 description 直译为中文，不做提炼总结。除非原始 description 特别长或同一项目包含多个子 skill description，否则不做整理归纳。
- README 表格不展示平台字段，统一按通用技能介绍；项目地址、Star 数、来源论坛分别独立成列。
- 来源论坛可使用 Linux.do favicon 图标：`https://cdn3.ldstatic.com/optimized/4X/c/c/d/ccd8c210609d498cbeb3d5201d4c259348447562_2_32x32.png`。
- GitHub README 不支持运行 JS 动态展示 star；可用 Shields.io 等外部图片徽章展示 GitHub 仓库和 star 数，但数值会有缓存延迟。
- README 中项目入口优先使用 GitHub 徽章和 star 徽章，增强在线阅读时的可扫读性。
- README 和静态页不展示 `status` 字段；正式收录条目默认按已核验终稿呈现。
- 结构化数据不再使用顶层 `status` 字段，改用 `verified_at` 和 `verification_sources` 记录核验信息。
- 单条 skill 的完整介绍放在 `docs/skills/`，详情页至少要让人快速看懂用途、适合场景、输入输出、核心特色、限制和来源。
- 静态页面入口为 `index.html`，用于后续发布到 GitHub Pages。
- 仓库结构：`data/skills/*.json` 保存结构化数据，`docs/skills/` 保存单条 skill 详情页，`index.html` 保存静态展示页面。
- 收录原则：只收录 Linux.do 社区用户发布的 skill；来源链接可以保留访问限制说明，例如 Linux.do `Lv1` 登录后可查看。
- 帖子原文只用于提炼 skill 的用途、特色、流程和局限性，不收录原文、不保存本地转录。
- `index.html` 不展示 Linux.do 来源权限提示，例如 Lv1、登录后可查看、可能需要权限；这些只保留在 JSON 或详情页。
- 静态页应优先高屏效比索引体验：卡片短、标签小、前置要求可扫读、限制只显示一句提醒，完整说明放详情页。
- 项目校验命令：`npm run check`。提交前必须运行，覆盖 JSON 解析、必填字段、README/静态页/详情页数量一致、本地链接、读者视图禁用词和内联 JS 语法。

## Maintenance Workflow

新增技能时按以下顺序执行：

1. 阅读 Linux.do 来源和原 GitHub 项目，优先确认 README、`SKILL.md`、示例、限制、许可证或特殊前置要求。
2. 新增 `data/skills/<id>.json`。字段至少包含 `id`、`name`、`author`、`repository`、`detail_page`、`verified_at`、`verification_sources`、`source`、`tags`、`one_liner`、`prerequisites`、`skill_description`、`use_cases`、`summary`、`workflow`、`limitations`、`notes`。
3. 新增 `docs/skills/<id>.md`。必须讲清：快速判断、适合场景、前置要求、输入输出、核心特色、和相近技能的差异、工作流程、当前限制、链接、备注。
4. 更新 `README.md` 技能清单和对应分类简介。README 面向最终读者，不写维护过程、不放帖子原文。
5. 更新 `index.html` 的 `skills` 数组、筛选标签和统计数字。不要在静态页展示来源权限提示。
6. 更新 `CLAUDE.md` 的条目记忆，尤其记录和相近技能的差异化定位、不可忽略的授权/商业限制、来源访问限制。
7. 更新 `CHANGELOG.md`。
8. 运行 `npm run check`，必要时再用浏览器打开 `index.html` 做冒烟检查。

发布或提交前检查：

- `npm run check`
- `git status --short`
- 确认没有 `.DS_Store`、`.playwright-mcp/`、截图、临时文件、帖子全文转录。
- 确认 README 和 `index.html` 不展示 `draft`、`status`、平台字段或 Linux.do 权限提示。
- 第一条收录是 `image-to-code-skill`，来源于 Linux.do 话题 `2314994` 和 GitHub 项目 `yuzhworkhard-wq/image-to-code`，内容基于原项目 README、SKILL.md 和用户提供的帖子原文综合提炼，但不保留原文存档。
- 原项目 README 和 SKILL.md 明确说明 `image-to-code-skill` 以 Codex skill 形式发布，但本索引读者视图按通用技能能力介绍，不展示平台字段。
- 第二条收录是 `guizang-ppt-skill`，来源于 Linux.do 话题 `2275626` 和 GitHub 项目 `op7418/guizang-ppt-skill`，内容基于 Linux.do 话题 JSON、原项目 README 与 SKILL.md 综合提炼，不保存帖子原文。
- `guizang-ppt-skill` 对外介绍重点应放在“单文件 HTML 横向翻页演示稿、电子杂志风、瑞士国际主义风、配图、截图风格化、多平台封面、版式自检”，不要写成普通 PPTX 模板。
- 第三条收录是 `awesome-ppt-skills`，来源于 Linux.do 话题 `2090997` 和 GitHub 项目 `stevenjinlong/awesome-ppt-skills`，内容基于 Linux.do 话题 JSON、原项目中文 README、`awesome-ppt`/`awesome-ppt-std`/`awesome-ppt-editable` 的 SKILL.md 综合提炼，不保存帖子原文。
- `awesome-ppt-skills` 对外介绍重点应放在“图片优先 PPTX、整页成图、标准版稳定输出、可编辑版实验性 ppt-master 重建”，并明确可编辑重建不是无损位图转矢量。
- 第四条收录是 `ppt-master`，来源于 Linux.do 话题 `1432693` 和 GitHub 项目 `hugohe3/ppt-master`，内容基于原项目 README、README_CN、`skills/ppt-master/SKILL.md` 和用户提供的来源链接综合提炼，不保存帖子原文。
- 用户提示 `ppt-master` 的 Linux.do 来源地址可能无法访问；展示层保留来源链接，详情页和结构化数据记录为可能需要登录或权限。
- `ppt-master` 对外介绍重点应放在“多格式文档输入、SVG 中间层、原生可编辑 PPTX、模板/品牌复用、实时预览、备注与动画导出”。它和 `awesome-ppt-skills` 的定位不同：`awesome-ppt-skills` 偏整页图片版 PPTX，`ppt-master` 偏原生可编辑 PPTX 工作流。
- 第五条收录是 `gpt-image2-ppt-skills`，来源于 Linux.do 话题 `2053515` 和 GitHub 项目 `JuneYaooo/gpt-image2-ppt-skills`，内容基于 Linux.do 话题、原项目 README、`SKILL.md`、实现逻辑和编辑能力说明综合提炼，不保存帖子原文。
- `gpt-image2-ppt-skills` 对外介绍重点应放在“gpt-image-2 整页视觉稿、图片型 PPTX、PPTX 模板克隆、自然语言指哪改哪、真实素材保真后贴、Markdown 编排和版本追踪”。它和 `awesome-ppt-skills` 都是图片优先路线，但本条更突出模板克隆、真实素材后贴和修改工作流；它和 `ppt-master` 的区别是后者偏原生可编辑 PPTX。
- 第六条收录是 `banana-slides`，来源于 Linux.do 话题 `1285413` 和 GitHub 项目 `Anionex/banana-slides`，内容基于用户提供的来源链接、原项目 README、`skills/banana-cli/SKILL.md` 和项目文档综合提炼，不保存帖子原文。
- `banana-slides` 对外介绍重点应放在“nano banana pro、Vibe PPT、一站式 Web 应用、素材工具箱、框选编辑、CLI、批量生成、PPTX/PDF/图片/讲解视频导出”。它和 `gpt-image2-ppt-skills` 都是图片生成驱动的 PPT 路线，但本条更像可部署产品；它和 `ppt-master` 的区别是后者偏原生可编辑 PPTX 工作流。
- Linux.do 话题 `1285413` 的 JSON 入口整理时不可公开访问；展示层保留来源链接，详情页和结构化数据记录为可能需要登录或权限。
- 第七条收录是 `thesis-defense-pptx-skill`，来源于 Linux.do 话题 `2091089` 和 GitHub 项目 `zouchenzhen/thesis-defense-pptx-skill`，内容基于 Linux.do 话题、原项目 README.zh-CN、README.en、`skills/thesis-defense-pptx/SKILL.md` 综合提炼，不保存帖子原文。
- `thesis-defense-pptx-skill` 对外介绍重点应放在“论文答辩专用、严格复用用户已有学校/学院/实验室 PPTX 模板、原生可编辑 PPTX、PowerPoint COM 复制模板页、逐页 PNG 导出、总览图检查、文字溢出检查和旧模板词扫描”。它和 `ppt-master` 都是原生可编辑 PPTX，但本条更窄且模板保真优先；它和图片型 PPT 技能的区别是输出可编辑对象而非整页图片。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jochne/linuxdo-awesome-skills](https://github.com/jochne/linuxdo-awesome-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
