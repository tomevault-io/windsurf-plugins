---
trigger: always_on
description: 本文件是 Codex、Claude Code 及其他贡献者共用的维护约定。[README.md](./README.md) 说明项目使用与发布，[CLAUDE.md](./CLAUDE.md) 提供 Claude Code 工作入口。变更架构或命令时同步更新相关文档。
---

# 项目维护指南

本文件是 Codex、Claude Code 及其他贡献者共用的维护约定。[README.md](./README.md) 说明项目使用与发布，[CLAUDE.md](./CLAUDE.md) 提供 Claude Code 工作入口。变更架构或命令时同步更新相关文档。

## 项目与工作范围

「大道总纲 · 段永平投资问答录」是整理公开投资问答、访谈、经营理念及公司人物资料的中文静态站点。技术栈为 Nuxt 4、Vue 3、Nuxt Content v3；站点发布产物在 `.output/public`。

- GitHub 仓库：`git@github.com:Esdeath/website_dao.git`，本地远程名 `origin`，主要分支 `main`。
- Node.js 使用 22.12 或以上版本，依赖以 `package-lock.json` 为准；部署工具使用 Git 和 Python 3。
- 开始修改前查看 `git status --short --branch` 与相关差异，保留工作区已有改动。
- 日常验证使用本地构建命令；提交、推送和部署按用户任务范围执行。
- `.env`、`raw/`、`.fontsrc/`、`node_modules/`、`.nuxt/`、`.output/`、`.data/` 及本地生成产物不入库，具体见 `.gitignore`。

## 命令与部署行为

```bash
npm ci                                                    # 安装锁定依赖
npm run dev                                               # 开发服务器
npm run typecheck                                         # 类型检查
node --test tests/*.test.mjs                               # 单元与集成测试
SKIP_DEPLOY=1 npm run generate                             # 本地静态生成
npm run preview                                           # 预览已有构建产物
npm run build                                             # Nuxt 构建
npm run og                                                # 更新 public/og-image.png
```

内容较多时使用：

```bash
SKIP_DEPLOY=1 NODE_OPTIONS=--max-old-space-size=8192 npm run generate
```

`npm run generate` 带有 `postgenerate: ./deploy.sh` 钩子；不设置 `SKIP_DEPLOY=1` 时，构建成功后会提交并推送。`npm run deploy -- "提交信息"` 与 `./deploy.sh "提交信息"` 直接执行相同部署入口，不重新构建。

部署实现由 `deploy.sh` 和 `scripts/deploy.py` 组成：

1. 切换到项目根目录，将 `.env` 中的设置导出到环境。
2. 远程优先取 `DEPLOY_GIT_REMOTE`；未指定时优先取 `github`，否则取首个远程。本项目当前为 `origin`。
3. 执行 `git add -A`，有暂存改动时提交，默认提交信息为 `chore: deploy`。
4. 执行 `git push -u <remote> <当前分支>`。脚本不自动拉取、变基或上传静态文件。

部署入口会暂存所有未忽略的改动，使用前检查改动归属。`SKIP_DEPLOY=1` 或非空的 `CI` / `CF_PAGES` 会跳过提交和推送；分离 HEAD 状态不支持部署。

Cloudflare Pages 可连接此仓库，使用 `main` 作为生产分支，构建命令为 `SKIP_DEPLOY=1 npm run generate`，输出目录为 `.output/public`。仓库连接、分支与构建状态以托管平台配置为准，不能仅凭 Git 推送成功认定站点已经上线。

## 环境变量

本地配置写在 `.env`，示例见 `.env.example`；正式托管环境在构建时提供相同变量。

| 变量 | 用途 |
| --- | --- |
| `NUXT_PUBLIC_SITE_URL` | 站点完整域名，用于 SEO、sitemap、robots 和链接；正式构建需填写 |
| `NUXT_PUBLIC_COMMENT_ENV_ID` | Twikoo 评论服务地址；留空关闭评论 |
| `DEPLOY_GIT_REMOTE` | 指定部署脚本使用的现有 Git 远程 |
| `SKIP_DEPLOY` | 设为 `1` 时跳过自动提交与推送 |

静态产物中的站点配置在构建时确定，修改变量后需要重新构建。`useArticleShareUrl.ts` 对未设置站点域名的本地环境有分享地址兜底；更换正式域名时一并检查此处。

## 内容模型与目录

`content.config.ts` 定义 `dao` 集合，类型为 `page`，读取 `content/dao/**/*.md`。

| 目录 | 内容 |
| --- | --- |
| `content/dao/concepts/` | 核心哲学、投资理念、企业经营、品格与心性、财务指标及推荐书单 |
| `content/dao/speeches/` | 访谈实录 |
| `content/dao/qanda/` | 四卷问答章节与主题总目录 |
| `content/dao/company-people/` | 公司、人物及相关专题档案 |

每篇 Markdown 的 frontmatter 必填 `title`、`slug`、`description`、`seoTitle`、`seoDescription`。可选字段为 `category`、`order`、`date`、`source`、`sourceUrl`、`sourceDate`、`type`、`tags`、`volume`、`volumeOrder`、`chapterOrder`，类型以 schema 为准。

维护约定：

- `slug` 必须全局唯一；站内交叉链接使用 `/<slug>`。路由依赖 slug，不依赖文件名。
- 为展示在目录中，填写合适的 `category` 和 `order`。新增分类同步 `app/pages/index.vue` 的 `daoCategoryOrder` / `categoryMeta`，以及 `app/layouts/default.vue` 的 `daoCategoryOrder`。
- 问答章节使用 `type: "qanda-chapter"` 和分卷字段。卷名为「投资原则与方法」「商业模式与经营」「公司案例」「人生与成长」。
- 问答总目录为 `content/dao/qanda/wenda-topic-index.md`，类型为 `topic-index`，访问路径为 `/wenda-topic-index`。首页与侧栏的问答数量只统计章节。
- 侧栏按 `volumeOrder` / `chapterOrder` 组织问答；文章上下章导航按全部问答章节的 `order` 排序，允许跨卷。
- 公司与人物侧栏按 `index`、`company`、`person`、`topic` 类型分组，新增档案参照同目录文章。
- 修改章节小节标题后，检查主题总目录的标题与锚点链接。改 URL 时同步维护 `public/_redirects`，避免旧地址失效或产生无效目标。

## 页面、查询与静态索引

页面入口为 `app/pages/index.vue` 和 `app/pages/[slug].vue`；阅读布局位于 `app/layouts/default.vue`。文章页通过 `queryCollection('dao').where('slug', '=', slug).first()` 获取内容，不存在时返回 404。

首页、侧栏和章节导航的列表查询只选所需元数据。尤其不要在 layout 的集合查询中选择 `body`：结果会序列化到每个页面的 payload，导致每次跳转重复加载全站正文。搜索全文使用独立索引。

`nuxt.config.ts` 开启链接爬取，并显式预渲染以下 Nitro 路由：

| 路径 | 作用 |
| --- | --- |
| `/sitemap.xml` | 首页和文章 URL；排除 `legacy-index` |
| `/robots.txt` | 爬虫规则及 sitemap 地址 |
| `/llms.txt` | 站点介绍与推荐阅读入口 |
| `/llms-full.txt` | 按分类列出文章标题、链接和摘要；排除 `legacy-index` 与 `topic-index` |
| `/search-index.json` | 带小节锚点的全文搜索数据；排除 `legacy-index` |

这些文件随静态构建发布，无需线上运行 Nitro 服务。`llms-full.txt` 是文章索引，不包含全部正文。新增类别时检查两份 LLM 索引的介绍和分类顺序是否需要同步。

`public/_redirects` 保留历史章节拆分、合并及旧入口页的 301 映射。旧入口页指向主题总目录，拆分或合并的文章指向对应有效章节；不要把所有旧 URL 统一替换为同一个目标。

## 搜索、锚点与阅读位置

- `LibrarySidebar.vue` 提供输入框与目录过滤，`SearchResultsPanel.vue` 将结果浮层 Teleport 到 `body`，固定定位在输入框下方。
- `useArticleSearchIndex.ts` 首次使用搜索时加载 `/search-index.json`；桌面侧栏和移动抽屉共享请求。索引不可用时退回元数据检索。
- 索引构建在 `server/utils/searchIndex.mjs`，检索、排序和片段生成在 `app/utils/searchSnippet.mjs`。这些纯函数保留 `.mjs`，便于直接使用 Node 测试。
- 索引结构为 `{ [slug]: { title, cat, type, v, n, tag, text, sections } }`。`text` 是小写并折叠空白的正文，`sections` 保存 `{ id, start, h }`。
- 小节 `id` 直接取 MDC/minimark AST 的 `props.id`，必须与页面标题 ID 一致。标题放在 `h` 中，不加入正文 `text`，以保持 `start` 偏移正确。
- 默认按标题、元数据、正文的优先顺序返回命中，每篇最多 4 条，总共最多 40 条；结果链接为 `/<slug>#<锚点>`。

正文滚动容器是 `.main-area`。`useArticleHashScroll.ts` 和 `app/utils/scrollToAnchor.ts` 负责等待正文挂载、定位及延迟校正。layout 切换路由时的回到顶部必须跳过带 hash 的链接。

搜索词通过 `sessionStorage` 的 `dyp:search-terms` 传给文章页，`highlightTerms.ts` 临时添加 `mark.search-hit`，约 4 秒后清除。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Esdeath/website_dao](https://github.com/Esdeath/website_dao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
