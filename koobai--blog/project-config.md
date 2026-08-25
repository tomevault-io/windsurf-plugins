---
trigger: always_on
description: 本文件是 AI 编程助手与自动化代理修改惊蛰项目时必须遵守的项目契约。开始任何修改前，应完整阅读本文件以及与任务直接相关的 `docs/` 文档。
---

# AGENTS.md

本文件是 AI 编程助手与自动化代理修改惊蛰项目时必须遵守的项目契约。开始任何修改前，应完整阅读本文件以及与任务直接相关的 `docs/` 文档。

## 项目目标

惊蛰是 Koobai 的真实生产博客，也是开放给他人使用的 Hugo 个人数字生活发布系统。

项目整理遵循最高优先级原则：

> 开源能力只做增量建设；Koobai 现有的内容、发布入口、数据同步、自动化和线上服务默认全部保持向后兼容。

## 当前状态

仓库同时承载 Koobai Production 和可复用的 Core/可选服务源码。当前生产代码包含 Koobai 专属配置；Core 初始化、统一检查和五个 Worker 开源部署包已经可用。任何 GitHub、Cloudflare 或生产服务的外部写操作仍需项目所有者单独授权。

## 源文件地图

- `config/_default/`：所有环境共享的 Hugo 结构和 Core 默认参数。
- `config/production/`：Koobai 生产身份、菜单、功能开关和公开服务配置。
- `config/development/`：保持 `hugo server` 与生产参考站一致的本地覆盖。
- `data/jingzhe/features.json`：机器可读功能注册表与数据职责目录；每份公开/私有 JSON 都声明写入者、读取者、生成属性和 Schema。
- `data/jingzhe/exercise.json`：运动名称、颜色、分组和食物换算的跨语言单一来源。
- `data/jingzhe/zouguo_boundary_catalog.json`：走过构建期完整行政区边界目录；浏览器只接收当前内容引用的裁剪子集，并须保留上游许可证。
- `data/exercise/activities.json`：来源无关的原始运动事实；只由同步网关写入，只由处理器读取。
- `jingzhe/exercise_contract.py`：Python 运动契约加载器，不得复制一套常量。
- `schemas/`：Front Matter、站点参数与公开 JSON Schema。
- `archetypes/`：新文章和新唠叨模板。
- `content/posts/`：长篇随笔。
- `content/laodao/YYYY/MM/`：短动态。
- `content/zouguo/`：独立走过 Markdown；带“走过”Tag 的唠叨和随笔仍以各自原文件为来源。
- `content/pages/`：独立页面和管理入口。
- `themes/jingzhe_v3/layouts/`：Hugo 全局与内容类型模板；`layouts/pages/` 只放 `content/pages/` 的专用页面布局。
- `themes/jingzhe_v3/assets/`：主题原生 CSS、JavaScript 与 Hugo Pipes 资源。
- `themes/jingzhe_v3/assets/css/style.css`：唯一全站样式入口；运动和走过分别通过各自目录中的 `index.css` 组合职责分片。
- `themes/jingzhe_v3/assets/js/navigation/`：灯箱、桌面导航、移动导航和共享滚动计算；构建时仍合并为 `js/site.js`。
- `themes/jingzhe_v3/assets/js/pages/`：普通页面交互脚本，经 Hugo Pipes 指纹化输出。
- `themes/jingzhe_v3/assets/js/zouguo/`、`layouts/zouguo.html` 与 `_partials/zouguo/`：走过地图、列表、统一 feed 和构建期边界裁剪。
- `themes/jingzhe_v3/assets/js/exercise/`：运动数据模型、日历 UI、隐私路线、Mapbox、海报与兼容控制器；构建时合并为一个脚本。
- `themes/jingzhe_v3/assets/js/pages/editor-core.js`：两个在线编辑器共享的鉴权、草稿、标签、上传、预览与 GitHub 原语。
- `themes/jingzhe_v3/assets/js/pages/editor-laodao.js`、`editor-post.js`：两个写作页面各自的 UI 与发布行为。
- `static/js/`：按上游许可证原样保留的第三方浏览器脚本。
- `assets/data/movies.json`：公开观影数据；由 `sync_movies.py` 原子更新。
- `assets/data/exercise/`：处理后运动、公共地标路线和月报等公开数据；App 与同步网关不得直接读写处理后的 `activities.json`。
- `process_activities.py`、`monthly_coach.py`：Actions 和旧调用方保持不变的兼容入口。
- `jingzhe/activity_processing.py`、`public_routes.py`：运动数据加工、展示字段和隐私公共路线。
- `jingzhe/monthly_stats.py`、`monthly_reports.py`：月报统计证据、模型调用、校验与冻结状态机。
- `sync_movies.py`：豆瓣观影数据同步。
- `tests/`：Python、浏览器脚本、Worker、文档和隐私契约测试。
- `workers/`：Publisher、Drafts、Comments、Likes、Activity Sync 五个独立 Cloudflare Worker，以及示例配置、D1 迁移和 OpenAPI。
- `.github/workflows/`：同步、处理、构建与部署流程。
- `docs/`：安装、架构、功能、配置、隐私和 AI 协作文档。

## 生成文件

以下目录或文件不是业务源码，不应手工编辑：

- `public/`
- `resources/`
- `.hugo_build.lock`
- `.hugo_build_lock`
- `__pycache__/`

若生成结果需要变化，应修改对应模板、原生 CSS、JavaScript、配置或数据源，然后重新构建。

## 生产兼容契约

除非用户明确授权迁移并提供回滚方案，否则不得破坏以下契约。

### 内容与 URL

- 保留 `content/posts/`。
- 保留 `content/laodao/YYYY/MM/`。
- 保留 `content/zouguo/`、`/zouguo/`、中文保留 Tag“走过”和内部类型 `zouguo`。
- 保留现有 Slug 和永久链接行为。
- 保留 `/newlaodao`、`/newsuibi`、`/movies`、`/exercise` 等入口。
- 不得批量重写历史内容 Front Matter。
- 不得为了修复已经停用的旧链接而批量修改历史文章或唠叨；优先在兼容层处理。
- 不得导致评论和点赞所使用的页面 URL 改变。
- 不得手工维护 `/zouguo/index.json` 或页面边界子集；它们必须由 Hugo 从三类 Markdown 来源生成。

### 网页写作

- 保留当前管理员 Token Header 和验证流程，直到 Publisher Worker 完成兼容迁移。
- 保留 GitHub 仓库写入路径、分支和 Front Matter 格式。
- 保留图片压缩和上传结果。
- 保留本地草稿、云草稿、最近内容编辑和标签补全。
- 保留现有 LocalStorage Key；如需迁移，必须读取旧 Key 并自动迁移。

### Actions 与自动化

- 不得随意改变 `Auto-sync activity facts`、`Auto-generate monthly coaching report` 等用于自动化识别和排障的提交信息。
- 运动处理工作流只监听 `main` 的原始事实文件，只能提交两个生成产物；部署工作流必须忽略仅包含原始事实的提交。
- 不得未经迁移同时更改现有 GitHub Secrets 名称。
- 不得让原始运动数据提交绕过现有处理步骤直接触发生产部署。
- 不得破坏豆瓣同步、运动处理和 Cloudflare Pages 部署。

### 动态服务

- Worker 源码位于 `workers/`，五个目录是独立最小权限边界，不得合并 Secrets。
- Activity Sync 只能写入环境中固定的 `data/exercise/activities.json`，不得接受客户端提供的仓库、分支或路径。
- Worker 路由、方法、Header、请求与响应字段变更必须先提供兼容层和测试环境。
- 当前生产 URL 不因源码入库而自动迁移；部署、域名切换和回滚由项目所有者决定。
- Comments 公开响应不得包含真实邮箱；只允许返回服务端生成的头像哈希。

### 运动与 AI 隐私

- 不得把私密轨迹、精确坐标、Polyline、`source_id` 或个人身份字段发送给模型。
- 隐私运动不得重新使用原始轨迹绘图。
- 公开轨迹与隐私替代路线必须继续明确区分。
- 修改 App/Gateway 上传字段或 `route_status` 时，必须同步 Schema、合成 Fixture 与 `tests/test_exercise_sync_contract.py`。
- 手机等设备本地视图首次接入必须使用 `delta`；只有权威全集连接器才能使用 `snapshot`。不得把单台设备暂时缺少的记录解释为删除。
- 月报必须基于程序生成的聚合证据，不得把模型当作医疗诊断工具。
- 修改隐私逻辑时必须增加或更新相应测试。

### 走过地点与来源

- Markdown 是走过正式内容唯一来源，来源类型固定为 `zouguo`、`laodao`、`post`。
- 走过时间使用 `zouguo.occurred_at`，不得用构建时间或最终发布时间替代。
- GeoJSON 和前端坐标顺序固定为 `[longitude, latitude]`；Front Matter 分别保存 `longitude`、`latitude`。
- 只能公开已经按 `privacy` 和 `precision` 确认可发布的地点；示例、测试和 Starter 不得复制真实家庭位置或 Koobai 的历史坐标。
- 删除独立走过可删除其 Markdown；唠叨和随笔应编辑原来源或移除“走过”Tag/地点块，不得误删原内容。
- 当前页面以地点为主，没有年份筛选。当前年日期显示 `MM-DD`，历史年份显示 `YYYY-MM-DD`。
- iOS App 不在本仓库；本仓库维护博客管线和 Worker 契约，不得在这里虚构或复制 App 源码。

## Secrets 规则

- 不得把私密 Token、密码、API Secret 或账户凭据写入 Git。
- 浏览器可公开的 Mapbox Token、Turnstile Site Key 与真正的 Secret 必须在文档中明确区分。
- 新增服务必须提供 `.env.example` 或等价示例，但示例值必须为空或明显无效。
- 日志、测试快照和错误输出不得包含 Secret。

## 个人内容规则

- `content/` 和 `assets/` 中包含真实个人内容与数据。
- 不得为了制作示例而复制家庭照片、真实坐标、健康数据或评论邮箱。
- Core 初始化内容和测试 fixtures 必须使用合成数据；不得为了模板复制真实内容。
- 不得擅自删除、改写或重新授权个人文章。

## 修改协议

1. 修改前检查 `git status --short --branch`，保留用户已有改动。
2. 确认任务模块和 Production 影响，避免无关的大规模改造。
3. 优先新增兼容层，再迁移旧实现。
4. 将高风险改动拆成可独立验证的小步骤。
5. 修改后运行与风险相称的检查。
6. 交付时说明行为变化、兼容性、测试结果和剩余风险。

## 基础验证

生产构建：

```bash
hugo --minify --panicOnWarning
```

Core 初始化回归：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koobai/blog](https://github.com/koobai/blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
