---
trigger: always_on
description: 自动化"语音/音乐/音频论文速递"流水线：arXiv + HuggingFace 抓取 → LLM 筛选 → 多模态深度分析 → 发布到 Hugo 博客 / 微信公众号 / 小红书 / 飞书。
---

# AGENTS.md

## 项目概述

自动化"语音/音乐/音频论文速递"流水线：arXiv + HuggingFace 抓取 → LLM 筛选 → 多模态深度分析 → 发布到 Hugo 博客 / 微信公众号 / 小红书 / 飞书。

**技术栈**：Node.js（核心流水线）+ Python（发布脚本）。要求 Node ≥ 18。`scripts/config.js` 集中管理 Node 端主要可调参数和当前运行数据文件路径（部分高频参数支持在项目根 `.env` 中用 `PD_*` 覆写）；`scripts/path_config.py` 集中管理 Python 发布/维护脚本共享路径。

详细执行规则见 `SKILL.md`，本文是紧凑版——只保留 Agent 不看代码就容易漏掉的要点。

## 常用命令

```bash
npm install              # 安装依赖（cheerio + pdf-parse）
npm test                 # 运行单元测试（node --test tests/*.test.js）
npm run fetch            # 全流程：抓取 + 筛选 + 深度分析
npm run deep             # 仅深度分析续跑（跳过已有 analysis；无分析结果时可从 filtered-papers.json 初始化）
npm run reanalyze        # 强制全量重分析（支持 --concurrency N）
npm run batch            # 批量分析未分析论文
npm run validate:data    # 只读校验候选/筛选/分析 JSON 数据、筛选决策缓存一致性和完整候选覆盖
npm run visual:post-publish -- --date YYYY-MM-DD # 在博客锁内建立 TOP 10 长图和汇总图任务
npm run visual:prepare -- --date YYYY-MM-DD # 校验 .bin 参考缓存并物化为带正确扩展名的内置生图输入
npm run visual:status -- --date YYYY-MM-DD # 只读校验 TOP 10 论文长图，不影响已经完成的博客发布
npm run cover:status -- --date YYYY-MM-DD  # 只读校验发布后的汇总图
npm run backfill         # 补录历史 paper ID（Python 脚本，不分析）
npm run blog:generate    # 只生成并安装 Hugo 博客文件
npm run blog:generate -- --date YYYY-MM-DD --exclude-id <arXiv ID>  # 明确排除单篇，可重复传入
npm run blog:review      # 只 review 已生成文件并保存 SHA-256 凭证
npm run blog:push        # 只验证凭证并 commit/push，不生成、不 review
npm run wechat           # 生成微信公众号草稿
npm run xiaohongshu      # 生成小红书文案
npm run visual:archive -- --date YYYY-MM-DD # 仅迁移旧版论文图片到日期/排行榜归档，不创建任务
npm run cover:archive -- --date YYYY-MM-DD  # 仅迁移旧版汇总封面到日期归档
npm run xhs-login        # 小红书登录（获取 Cookie）
npm run xhs-publish      # 小红书自动发布单篇
npm run xhs-publish-all  # 小红书自动发布全部

# 直接调用（不在 package.json 中）
node scripts/quick-test.js              # 快速测试（抓+筛选，不分析）
node scripts/analyze-single-paper.js <arxiv-id> [--force]  # 单独分析一篇论文（--force 覆盖已有结果）
node scripts/reanalyze-selected.js <arxivId1> [arxivId2] ...  # 重分析指定论文
node scripts/refilter-reanalyze-by-date.js <date>  # 按日期重新筛选+分析
node scripts/validate-scores.js         # 验证并修复评分
node scripts/test-api-key.js            # 测试 LLM API key 可用性
python3 scripts/publish-to-feishu.py    # 生成飞书文档

# ICML 2026 专属流程（仅 icml-2026-analysis 分支可用）
npm run icml-fetch-openreview   # 从 OpenReview API 抓取论文元数据（需 Chrome Cookie）
npm run icml-filter             # LLM 筛选音频/语音/音乐相关论文
npm run icml-download-pdfs      # 下载筛选论文 PDF 并提取文本（含表格）
npm run icml-analyze            # 批量深度分析（基于 PDF 全文 + 自动注入图片）
npm run icml-retry              # 重试失败的分析
npm run icml-reanalyze-pdf      # 基于 PDF 全文重分析
python3 scripts/extract-icml-images.py   # 提取 PDF 图片到图床
# 会议博客也须依次运行 generate-blog.py、review-blog.py、push-blog.py；生成阶段传 category/date/data file
```

未配置 linter、typecheck 或 formatter。CI 会运行 `npm test`、`npm run validate:data`、所有 `scripts/` / `tests/` 下 JS 文件 `node -c` 语法检查、所有 `scripts/` 下 Python 文件 `py_compile` 语法检查、`tests/python` 下 Python 单测，以及所有 `.sh` 的 `bash -n`。

## 环境配置

复制 `env.example` → `.env`（已 gitignore）。

**抓取/筛选/分析必需变量**：`PAPER_ANALYZER_API_KEY` / `PAPER_ANALYZER_MODEL` / `PAPER_ANALYZER_ENDPOINT`

`PD_ANALYSIS_REPAIR_MAX_TOKENS` 控制审校重写、表格补充、方法补充与结构修复的输出上限，默认 16000；主分析仍保留 64000 上限，避免局部修复在推理模型上长期思考并撞到供应商网关超时。

**博客相关变量**：`PAPER_DIGEST_BLOG_REPO` 可覆写 Hugo 博客仓库路径；未设置时使用默认路径，目录不存在会跳过博客已发布去重，真实博客发布仍需要本地仓库存在。
`PD_BLOG_REVIEW_CONCURRENCY` 控制博客独立论文页的三层 review 并发度，默认为 5；汇总页仍先完成审查。首次 review 失败会保存绑定生成清单、博客 `main` 基线和逐文件 SHA-256 的失败集状态；修复后仅复审已修改的失败文件，已通过文件或基线发生变化时自动退回全量 review。
`PD_XIAOHONGSHU_ONELINER_CONCURRENCY` 控制小红书 TOP N 一句话亮点的 LLM 并发度，默认 5，限制为 1–5；结果必须按原排名回填，单篇失败独立使用本地摘要回退。

**双模型（可选，多模态）**：设置 `PAPER_ANALYZER_SECONDARY_MODEL` 即启用副模型做图像筛选与插图计划（主模型仅做纯文本分析）；副模型只输出 JSON 计划，包含目标章节、代码提供的稳定 `paragraph_id`、图前 `lead` 和图后 `explanation`。代码只新增插图及相邻说明，忽略旧 `replacement` / `rewrite` 字段，禁止副模型替换主模型原文；每篇默认最多插入 4 张，非法段落 ID 直接丢弃，旧 `anchor` 格式仅保留兼容。不设置则跳过图片、退回单模型纯文本。`PAPER_ANALYZER_SECONDARY_ENDPOINT` / `PAPER_ANALYZER_SECONDARY_API_KEY` 未设置时默认复用主模型对应值。

**Codex 发布后视觉资产**：必须先完成全部论文深度分析，再依次完成博客 generate、全量/失败集 review、push，并由 `push-blog.py` 验证远端 `main` OID。只有发布凭证同时记录 `publicationCommit`、相同的 `remoteVerifiedOid` 和验证时间后，才可建立视觉任务。`push-blog.py` 会自动运行发布后规划器；论文长图只选择最终评分 TOP 10（同分按规范化 arXiv ID），每篇一张纵向 PNG，顶部为完整英文标题、正文为简体中文；另生成一张包含批次标题、热门方向和 TOP 10 排行榜的汇总图。默认成品必须由 Codex 内置 `image_gen` 一次性生成完整带字构图，让标题、中文说明、结构关系、实验数字、论文关键图与纸张拼贴视觉自然配合；禁止再经过确定性文字卡片合成器。Agent 必须逐图目视核对标题、中文正文、箭头/并行分支、指标方向、论文数字与排行榜后才能执行 record，不可读或存在实质错误时必须重生成。两类 manifest 按日期隔离并支持失败项续跑，但图片不进入本轮博客 generation/review/push，也不阻断已经完成的发布。项目脚本不得调用图像 API或读取 `OPENAI_API_KEY`，实际绘图只能由 Codex 内置 `image_gen` 完成；`render-visual-summary.py` 仅用于本地调试和离线兜底。

Node 脚本通过 `scripts/env-loader.js` 加载当前项目根 `.env`：① `scripts/config.js` 模块级最先执行（任何 `require('config')` 即触发）；② `scripts/utils.js` 的 `loadEnvFile()` 二次兜底补漏。Python 脚本通过 `scripts/project_env.py` 加载同一个 `.env`。两端都会先清理继承自 Trae/Codex/shell 的项目同名变量（`PAPER_ANALYZER_*`、`PAPER_DIGEST_*`、`PD_*`、`WECHAT_*`、`FEISHU_*`、`XIAOHONGSHU_*`、`KIMI_API_KEY`）以及大小写代理变量，再写入项目 `.env`；代理不再回退 macOS `scutil`，加载器会把 `.env` 权限收紧为 `0600`。外部命令必须使用最小子进程环境，禁止把 LLM/发布凭据传给 curl、CLI、Git hook 或浏览器进程。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanless/audio-paper-digest](https://github.com/nanless/audio-paper-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
