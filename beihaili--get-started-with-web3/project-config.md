---
trigger: always_on
description: 默认用中文交流。错误信息可保留英文，分析和说明尽量用中文。
---

# Get Started with Web3 Agent Notes

默认用中文交流。错误信息可保留英文，分析和说明尽量用中文。

## 项目定位

这是一个 Web3 学习平台，包含 Markdown 教程内容、React SPA、AI Tutor、徽章系统、SEO 构建脚本、捐赠/赞助配置，以及面向 AI Agent 的只读内容接口。当前主课程由 `src/config/courseData.js` 驱动，覆盖 Web3 快速入门、比特币技术、以太坊与智能账户、Web3 Builder、DeFi、Layer 2/跨链、DAO 等模块。

## CEO 运营目标

项目当前运营目标是把 Get Started with Web3 做成具备持续增长、持续运营和收入能力的开源 Web3 学习平台。核心里程碑是达到 1000 GitHub stars，并让项目同时提升 beihai 的个人影响力。

运营文档：

- `docs/strategy/2026-05-14-ceo-operating-system.md`: CEO 操作系统，包含使命、北极星指标、增长闭环、变现模型、30/60/90 天路线图和外部动作审批边界。
- `docs/strategy/2026-05-14-execution-board.md`: 执行看板，包含 KPI、工作流、前 10 个任务、内容发布队列和赞助线索。
- `docs/strategy/2026-05-14-sponsor-kit.md`: 赞助包草案，包含受众、价格、曝光权益、接受政策和月度汇报指标。
- `docs/strategy/2026-05-14-sponsor-outreach-drafts.md`: 赞助外联话术草案；常规外联可自主执行，高风险赞助对象仍需单独确认。
- `docs/strategy/2026-05-15-sponsor-leads-tracker.md`: 具名赞助/资助线索 tracker，记录公开来源、fit score、建议外联角度、渠道、状态和风险边界；发送外联后必须同步每日运营汇报。
- `docs/strategy/2026-05-18-donation-affiliate-disclosure-review.md`: 捐赠、赞助、联盟链接和加密货币打赏的披露审查记录；新增或扩大 affiliate、payment、wallet、x402 相关入口前必须保持公开披露并确认风险边界。
- `docs/strategy/2026-05-18-safe-reown-impact-memo.md`: 面向 Safe Ecosystem Foundation 和 Reown / WalletConnect 的资助/赞助 impact memo 草案，包含当前指标、官方来源、建议 ask、外联话术和发送前检查清单；尚未发送外联。
- `docs/strategy/2026-05-18-paid-ai-tools-landing-copy.md`: 未来 hosted AI 工具 landing copy 草案，覆盖 `generate_personalized_web3_plan` 和 `audit_learning_answer` 的定位、CTA、边界文案、FAQ 和 launch checklist；不得描述为已上线付费能力。
- `docs/strategy/2026-05-14-awesome-list-submissions.md`: awesome-list 和社区分发追踪，包含定位文案、目标列表、PR 模板和中文社区帖草案。
- `docs/community/contributor-ladder.md`: 贡献者成长路径，定义 first-time contributor、repeat contributor、reviewer、module steward 和 community/sponsor ally。
- `docs/community/contributor-spotlight-template.md`: 月度贡献者 spotlight 模板，用于把公开 PR、issue、社区分发和增长指标转化为 GitHub/社交更新；不能承诺代币奖励、付费曝光或金融背书。
- `docs/community/spotlights/`: 已发布贡献者 spotlight 目录；只引用公开 GitHub username、PR、issue、公开指标和贡献影响，不写未经同意的个人信息。
- `docs/community/good-first-issues.md`: 可复制到 GitHub Issues 的 starter issue 清单，包含标签、背景、验收标准和验证方式。
- `docs/operations/`: 每日运营汇报目录；`README.md` 定义 cadence、数据源和风险边界，`templates/daily-report-template.md` 是每日汇报模板，按 `YYYY-MM-DD-daily-report.md` 记录实际进展、指标、部署、外部分发、赞助线索和下一步。

社区入口：

- `CONTRIBUTING.md` / `CONTRIBUTING.en.md`: 双语贡献指南，维护贡献路径、验证矩阵和内容质量标准。
- `.github/ISSUE_TEMPLATE/`: Bug、feature、content/translation、growth/community 分流模板。
- `.github/PULL_REQUEST_TEMPLATE.md`: PR 验证和内容安全检查清单。
- `.github/workflows/deploy.yml`: PR 构建会上传 `dist` artifact；同仓库 PR 会用固定 marker 更新同一条 PR 评论，fork PR 因 `GITHUB_TOKEN` 写权限受限会跳过评论但不应让构建失败；合并到 `main` 后才部署 GitHub Pages。
- `.github/dependabot.yml`: 每周检查 npm 依赖和每月检查 GitHub Actions；minor/patch 更新分组，major 更新单独处理；`@commitlint/cli` v21、`@commitlint/config-conventional` v21 和 `lint-staged` v17 当前被忽略，因为它们要求 Node 22，而项目 CI 仍使用 Node 20，需等计划内 Node 22 迁移后再解除。
- GitHub Issue [#156](https://github.com/beihaili/Get-Started-with-Web3/issues/156): 已 pin 的公开 1000-star roadmap；维护 starter issue 队列、翻译、分发、AI-native 和赞助动作时同步参考。

工作边界：

- 可直接修改仓库内的战略文档、roadmap、运营文案、issue 模板和增长相关代码。
- beihai 已授权 Codex 自主批准并执行常规外部动作，包括发起增长文案草稿、联系赞助商、修改 GitHub repo 描述/topics、推送工作分支、创建 PR 和发布项目运营材料；执行后在每日汇报中留痕。
- 高风险或不可逆动作仍需单独确认：修改收款地址、接受高风险金融/交易类赞助、删除仓库、私有化仓库、转移仓库所有权、泄露 secrets、绕过安全检查、直接推送 `main` 导致生产发布。

## 常用命令

```bash
npm run dev             # 本地开发
npm run build           # 生产构建
npm test                # 全量测试
npm run lint            # ESLint 检查
npm run ai:index        # 生成 AI-native 内容索引
npm run ai:publish      # 复制 AI artifacts 到 public/，供 GitHub Pages 发布
npm run ai:verify       # 验证公开 AI 入口、MCP 工具清单和 x402 元数据
npm run translation:check # 非阻塞检查 zh/ 中缺失的英文翻译
npm run mcp:web3        # 启动本地 stdio MCP server
```

## AI-Native 内容层

- `ai/manifest.json`: 服务清单，包含仓库信息、artifact 路径、MCP 命令、可复制的 `mcpServers` client config 示例和未来 x402 工具元数据。
- `ai/content-index.json`: 双语课程和术语表索引，供 Agent 搜索、引用和组合上下文。
- `ai/llms.txt`: 面向 Agent/crawler 的文本入口。
- `public/llms.txt`: 自定义域名根路径公开入口，部署后为 `https://bhbtc.xyz/llms.txt`。
- `public/ai/manifest.json`: 部署后为 `https://bhbtc.xyz/ai/manifest.json`。
- `public/ai/content-index.json`: 部署后为 `https://bhbtc.xyz/ai/content-index.json`。
- `scripts/generate-ai-index.mjs`: 生成上述 artifact。
- `scripts/publish-ai-artifacts.mjs`: 将根目录 `ai/` artifacts 复制到 `public/`。
- `scripts/verify-ai-entrypoints.mjs`: 检查 artifacts、公开 URL、MCP 工具清单、中英文覆盖和 x402 元数据。
- `scripts/sync-content.mjs`: 将中英文课程内容复制到 `public/content/`，为源目录中的 `README.MD` 生成发布用规范别名 `README.md`，为英文课程补齐对应中文课程中存在但英文目录缺失的本地图片资产，并生成 `public/content/image-metadata.json`，供课程图片渲染时补充 `width` / `height`。
- `scripts/check-translation-coverage.mjs`: 非阻塞检查 `zh/` 中缺失的英文翻译，支持 `README.md` 和 `README.MD` 两种文件名。
- `scripts/ai-content-core.mjs`: 搜索、读取课程、生成学习路径、组合上下文等纯函数。

修改课程结构、术语表或 Agent 工具元数据后，运行 `npm run ai:index && npm run ai:publish && npm run ai:verify`，并提交更新后的 `ai/` 与 `public/` artifacts。

## 官网 URL、域名与统计

- `src/config/siteConfig.js`: 统一管理默认官网 URL、base path、canonical/share URL 拼接；默认生产官网为 `https://bhbtc.xyz`，默认 base path 为 `/`。
- GitHub Pages 自定义域名已开始切换：`public/CNAME` 为 `bhbtc.xyz`，CI 构建显式使用 `VITE_SITE_BASE_URL=https://bhbtc.xyz`、`SITE_BASE_URL=https://bhbtc.xyz` 和 `VITE_BASE_PATH=/`。如需临时回到仓库路径预览，可设 `VITE_BASE_PATH=/Get-Started-with-Web3/`。
- `index.html` 保留 GA4 和 Cloudflare Web Analytics 脚本；GA 自动初始 pageview 已关闭，由 `src/components/RouteAnalytics.jsx` + `src/utils/analytics.js` 在 SPA 路由切换时发送 `page_view`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beihaili/Get-Started-with-Web3](https://github.com/beihaili/Get-Started-with-Web3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
