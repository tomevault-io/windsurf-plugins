---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目是什么

一个 MCP 服务（TypeScript，`@modelcontextprotocol/sdk`），把 Google Search Console、Google Analytics 4（Data API 与只读 Admin API）、网页与 GEO 审计（PageSpeed、CrUX、结构化数据、AI 爬虫、llms.txt 等）、跨数据源分析，以及可选的 WordPress（通过 SSH 执行 WP-CLI）和 GitHub 读写封装成约 80 个工具，用于 SEO/GEO 运维。生产环境是部署在服务器上的 Streamable HTTP 实例，所有客户端都连它；本机 stdio 只用于开发验证。面向用户的说明在 `README.md`（英文）和 `README.zh-CN.md`（中文）。

## 常用命令

```bash
npm run build          # tsc 编译到 dist/（postbuild 会给 dist/index.js 加执行权限）。每次改 src 后必须重新构建：客户端跑的是 dist/，不是 src/
npm run dev            # tsx src/index.ts（stdio，免构建）
npm start              # node dist/index.js（stdio）
npm run start:http     # node dist/index.js --http（或设置 MCP_TRANSPORT=http）
npm run inspector      # 用 MCP Inspector 调试 dist/
npm run docs:sync      # 按工具清单快照同步两份 README 与 package.json 的工具计数（npm test 会校验）
npm run check:secrets  # 扫描所有已跟踪文件里的密钥与个人信息（提交/推送钩子会自动跑）
npm run auth -- --client-secret ./client_secret.json   # 一次性 OAuth 授权，写入 ~/.config/google-seo-mcp/credentials.json
```

`npm test` 依次跑：`test/unit/*.test.mjs`（`node:test`，针对 `dist/` 里导出的纯函数：robots 解析、URL/路径归一化、日期、dotenv、schema 审计、工具分类）、`test/smoke.mjs`（启动服务、检查描述与注解、比对 `test/tools.snap.json`，增删工具后用 `UPDATE_SNAPSHOT=1 npm test` 刷新）、`scripts/sync-readme.mjs --check`。全部不访问网络。CI（`.github/workflows/deploy.yml` 的 test 任务）在每次推送和 PR 上跑同样的东西加密钥扫描，main 只有在它通过后才部署。真实调用的验证用临时的 MCP 客户端脚本：

```js
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";
const c = new Client({ name: "t", version: "0" });
await c.connect(new StdioClientTransport({ command: "node", args: ["dist/index.js"], env: { ...process.env, GOOGLE_APPLICATION_CREDENTIALS: "service-account.json", WP_SITES: "[...]" } }));
console.log(await c.callTool({ name: "gsc_list_sites", arguments: {} }));
```

脚本要放在仓库根目录运行（需要从 `node_modules` 解析 SDK），用完删掉。测 HTTP 模式用 `curl -X POST /mcp`，带 `Authorization: Bearer` 和 `Accept: application/json, text/event-stream` 两个头。

**写入类工具的测试只能作用于临时对象**：WordPress 用 `wp post create --post_status=draft` 建的草稿（测完 `--force` 删除）、临时重定向（建了就删）、与当前值相同的无变化写入；GitHub 用临时分支（`createBranch`，测完删分支）；Search Console 只重新提交已有站点地图。绝不在真实文章、真实分支上做测试写入。

## 架构

- `src/server.ts`：`createServer()` 创建 `McpServer` 并注册全部工具。两种传输都调用它；HTTP 传输是**每个请求新建一个服务实例**（无状态，`sessionIdGenerator: undefined`）。它包了一层 `registerTool`：按工具名推断注解（`WRITE_TOOLS`、`DESTRUCTIVE_TOOLS` 正则）、只读模式下跳过写入工具、按 `toolsetOf()` 应用工具集筛选。**新增写入类工具时必须让名字匹配这两个正则**，否则会被当成只读。服务器 instructions 在 `buildInstructions()` 里。
- `src/index.ts`：入口，根据 `--http` 参数或 `MCP_TRANSPORT=http` 选择 stdio 或 HTTP。`src/http.ts` 是纯 `node:http` 服务，带 Bearer Token 鉴权（`MCP_AUTH_TOKEN`）、`/healthz`（无令牌只返回 `{ok:true}`，带令牌附版本号与凭据来源），默认只绑回环地址。
- `src/google.ts`：单例 `GoogleAuth`，以及 `googleapis` 客户端工厂（`searchconsole v1`、`analyticsdata v1beta`、`analyticsadmin v1beta`）。凭据查找顺序：`GOOGLE_CREDENTIALS_JSON` → `GOOGLE_APPLICATION_CREDENTIALS` → `~/.config/google-seo-mcp/credentials.json` → ADC。GA4 用的是 `googleapis` 的 REST 客户端而不是 `@google-analytics/data`，避免引入 gRPC。
- `src/util.ts`：`tool(fn)` 包装所有处理函数，返回值经 `fitResult()` 做体积保护（超过 `SEO_MCP_MAX_RESULT_CHARS` 时对最长的数组减半直到放下，并加 `_truncated` 说明）后以**紧凑 JSON**（不缩进）写进文本内容；抛出的异常经 `formatError` 变成 `isError` 结果（缺凭据和 403 会附加提示）。`heartbeat(extra, msg)` 给长任务发进度通知，超过约 10 秒的工具都要用。`resolveDate()` 把 `today`、`yesterday`、`NdaysAgo` 转成 `YYYY-MM-DD`，因为 Search Console 只接受绝对日期。
- `src/tools/gsc.ts`、`src/tools/ga.ts`：Google 工具。输入 schema 是传给 `registerTool` 的 zod raw shape，`.describe()` 文本要写清楚，那是 LLM 唯一能看到的说明。GA 的行数据由 `tabulate()` 拍平成 `{维度: 值, 指标: 数字}` 对象。
  - `gsc.ts` 导出 `query()`、`normalizePath()` 供其他模块复用；`gsc_delete_*`、`gsc_add_site` 是写入工具，名字必须保持这些前缀。
  - `ga.ts`：漏斗报告走 v1alpha，`googleapis` 没封装，用 `getAuth().getClient().request()` 直接 POST；漏斗步骤里页面条件要用 `unifiedPagePathScreen`（`pagePath` 不被接受），返回的 `metricHeaders` 会重复一遍，按名字去重后再对应 `metricValues`。`ga_property_config` 混用 admin v1beta 和 v1alpha（受众、增强型衡量只在 alpha）。`ga_check_compatibility` 在组合本身不兼容时 API 返回 400 而不是列表，已捕获成 `compatible:false`。
- `src/tools/web.ts`：不依赖 Google 授权的网页检查（`page_audit` 用 cheerio 解析、`pagespeed`、`sitemap_check`、`robots_check`）。`collectSitemapUrls()` 和 `parseRobots()` 被 gsc 模块复用。
- `src/tools/crawl.ts`：`site_crawl`（去重用去尾斜杠的 key，但请求始终用原始 URL，否则会误报 301 链）、`hreflang_check`、`compare_pages`（识别反爬页）、`social_preview_check`、`keyword_suggest`。
- `src/tools/analysis.ts`：跨数据源分析（`migration_check`、`cross_site_links`、`content_refresh_candidates`、`knowledge_graph_check`、`crux_history`、`brand_mentions`、`reviews_snapshot`），依赖 `gsc.ts` 导出的 `query()` 和 `normalizePath()`。
- `src/tools/github.ts`：GitHub REST，`github_commit_files` 用 Git Data API 一次提交多文件；token 取 `GITHUB_TOKEN`，否则 `gh auth token`。
- `src/tools/geo.ts`：GEO 与信任信号检查。`BOTS` 表维护 AI 爬虫的 robots 令牌和 UA 字符串；`SCHEMA_RULES` 是各 schema 类型的必填/推荐字段表；`analyzePage()` 是 `geo_page_score` 和 `eeat_audit` 共用的页面信号提取。`indexnow_submit` 和 `ai_citation_check` 依赖可选环境变量，缺失时返回带说明的错误而不是不注册。
- `src/tools/wp.ts`：WordPress 工具。只在设置了 `WP_SITES`（JSON 数组）或 `WP_SSH_*` 环境变量时注册。每次调用都是 `spawn` 一个 `ssh … 'cd <path> && wp …'`；所有远程参数都经 `shq()` 做 POSIX 单引号转义。大块数据（正文、构建器修改）通过 stdin 传，不放进 argv。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Akxan/google-seo-mcp](https://github.com/Akxan/google-seo-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
