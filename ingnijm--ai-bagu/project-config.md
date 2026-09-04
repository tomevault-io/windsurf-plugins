---
trigger: always_on
description: 本文件只写本仓库特有约定。通用协作规范见用户全局 `AGENTS.md`。
---

# 八股助手（八股抽问）— 项目 Agent 规则

本文件只写本仓库特有约定。通用协作规范见用户全局 `AGENTS.md`。

用户若提到 `AGENT.md`，即本文件（标准文件名 `AGENTS.md`）。

## 项目是什么

本地面试复习工具：`bagu.py` 单文件核心 + `web/index.html` 唯一页面，另有 `android/` 原生宿主。SQLite 存题、复习进度及已完成的评分结果。桌面 CLI / 网页共用数据库和**同一把会话锁**；Android 复用核心，但使用独立私有目录，不读取电脑题库或配置。Hermes 聊天走 CLI 并自行评卷；桌面网页和 Android 可用配置的 OpenAI 兼容模型评卷。

## 目录

| 路径 | 职责 |
| --- | --- |
| `bagu.py` | CLI、SQLite、会话、抓题、HTTP、评卷、设置、备份与运行路径注入 |
| `web/index.html` | 桌面和 Android 共用 UI（答题/背题 + 配置库 + 题库管理） |
| `android/` | WebView 宿主、原生桥接、Python 启动层、Java/仪器测试 |
| `assets/` | 品牌、离线字体及许可证、设计参考；打包仅用显式允许列表 |
| `scripts/` | Android 构建/校验与清洁题库种子生成 |
| `test/test_bagu.py` | 单元测试，必须用临时目录，禁止写真实 `bagu.db` |
| `test/test_android_project.py` | Android 项目、运行时、原生桥接和发布契约测试 |
| `docs/README.md`、`docs/user-guide.md` | 文档导航与详细使用说明；根 README 面向首次使用 |
| `docs/cli.md`、`docs/api.md`、`docs/architecture.md` | 命令、接口与当前架构/数据约束 |
| `docs/development.md`、`docs/android-beta.md` | 开发测试、Android 构建与安装更新 |
| `docs/data-transfer-and-updates.md` | 双端迁移、Android 更新、版本化构建与明确授权的 GitHub 发布／恢复 |
| `docs/validation.md`、`docs/images/` | 历史验收证据与文档截图；截图不加入应用打包列表 |
| `docs/superpowers/specs/` | 已定设计。会话网页、多模型条目均已实现 |
| `docs/superpowers/plans/` | 实现计划 |
| `.env` | 密钥，禁止提交、禁止写入文档/聊天 |
| `settings.json` | 非密钥配置，禁止提交 |
| `bagu.db` | 本地题库，禁止提交 |
| `.signing/`、`dist/` | 本地签名材料、生成的交付物，禁止提交 |

不要新增第三方 Python 依赖。不要把 HTTP 绑到非本机地址。不要新开第二个 HTML 文件（配置也在 `web/index.html`）。

## 不可违反

1. **会话锁**：每份数据库最多 1 条 `sessions.status = 'open'`，由部分唯一索引保证。有 open 会话时 `draw` 必须失败，不创建新会话。
2. **一次评分**：`grade(session_id, qid, result)` 同一会话同一题只认第一次；重复 / 错会话 / 题不在本轮 → 失败且不改库。
3. **skip 不调度**：只把会话标 `closed`，未判题的 `next_due` / `level` / `times_seen` 一律不动。
4. **CLI grade 必须带 session**：`grade <session_id> <id> <again|hard|good|easy>`。旧两位参数已废除。
5. **失败不落库**：模型 HTTP / 断流 / 解析失败不得评分；评分返回结果及答案 HTML 必须在事务提交前构造，渲染异常回滚调度、grade 与 submission。
6. **密钥**：Key 只在 `.env`。`settings.json` 禁止写 Key。GET 接口只回 `api_key_masked`。禁止把真实 Key 写进源码、测试（测试用 `sk-test` 这类假值）、README、commit、日志。
7. **禁止拷贝 Nous OAuth** 进本项目。
8. **Hermes 路径不调本仓库 LLM**：`bagu.py` 的 CLI `grade` 只落库；评卷由 Hermes 自己完成。桌面网页/Android 才走 `_openai_chat` / `_openai_chat_stream`。
9. **重放不再计分**：网页/Android 的同一 submission、同一会话和题目重试只返回已存结果；不同 submission 对已评分题仍失败，跨题复用 ID 失败。
10. **移动端隔离**：Android 仅监听 `127.0.0.1` 随机端口，API 校验每进程令牌；模型地址仅允许 HTTPS。不得放宽 WebView、跨源重定向、原生存储或文件选择边界。

## 当前实现（以代码为准）

设置是**多模型条目**：

- `settings.json`：`{active_id, models:[{id,name,provider,model,base_url}]}`
- `.env`：`BAGU_KEY_<id>=...`（每条模型一把钥匙）
- HTTP：`GET/POST /api/models`、`POST /api/models/test`、`PUT /api/models/:id`、`POST .../activate|copy`、`DELETE /api/models/:id`
- `GET /api/settings` 只读当前 active（兼容）；`POST /api/settings*` 已 404
- 网页：作答页顶部当前模型条 → 配置库选用/新建/修改/复制/删除；无 Hermes 导入
- 新建/修改及 `/api/models/test` 使用完整流式响应验证，不只收到首个 chunk 就判成功；激活/复制不重新测试
- 同步和流式请求共用构造/解析规则，默认不传 `temperature`；截断、拒答、空输出或不完整流不会计分

`load_settings` 返回 `models` / `active_id`，并把 active 的 `provider/model/base_url/api_key` 提到顶层给评卷用。

面经题包与专题模拟也已在当前源码实现：

- `.bagu-pack` 仅支持用户本地显式检查、确认安装和更高 revision 增量升级；题包题只读，默认可加入日常复习，可关闭但不物理卸载
- 练习入口区分日常复习与面经模拟；专题可按整套或章节启动，技术题沿用评分/调度，`prepare` 准备题只允许 `prepared|skipped` 完成且不调用模型、不改调度
- 桌面和 Android 都先检查再确认同一份字节；Android 原生持有正文，JS 只接收允许列表结果。没有在线商店、自动下载、自动更新或公开首包
- 仓库、public/internal 种子与公开 APK 不内置真实题包；源面经、私有审校清单和生成的 `.bagu-pack` 不提交、不公开，也不从历史审计基线自动迁移

## 相关设计

`docs/superpowers/specs/2026-08-26-model-profiles-design.md`（已实现）。

会话网页 spec：`docs/superpowers/specs/2026-08-26-session-web-design.md`（已实现）。改会话协议前先读它。

后续补充：`docs/superpowers/specs/2026-08-27-session-fault-recovery-design.md`（数据库并发、submission 与中断恢复）和 `docs/superpowers/specs/2026-08-27-android-beta-design.md`（移动端边界）。早期计划是历史记录，部分旧设计已被替代；阅读入口为 `docs/README.md`。当前使用方法见 README / `docs/user-guide.md`，技术说明见 CLI / API / architecture 文档，均需结合各文档注明的源码基线和实际代码核对。测试数字、APK 哈希与未验证范围集中在 `docs/validation.md`，不作为后续版本的自动验收结论。

## 数据表

`questions`：除题干、分类、答案、URL 和原有调度字段外，新增 `pack_id`、`stable_question_id`、`question_type=review|prepare`、`preparation_prompt`、`answer_review_status`、`retired`。本地题用部分唯一索引 `(category, question)`；题包题用 `(pack_id, stable_question_id)`，因此不同面经可保存相同文本并独立计算进度。题包题只读，升级按稳定 ID 更新内容并保留主键和进度。

`question_packs` 保存稳定 `pack_id`、revision、显示版本、源/内容哈希、manifest 声明数量和 `include_in_review` 本地偏好；`question_sources` 保存题包题的有序来源。`experiences`、`experience_sections`、`experience_items` 保存 `interview|topic_set` 专题、章节及有序题目关系。

`sessions`：`id TEXT PK`，`status` 仅 `open|closed`，`session_type=review|experience`，并可保存专题/章节上下文；每库仍最多一条 open。

`session_items`：`(session_id, question_id)` PK，`position` 保存专题原顺序，`completion_type` 仅 `graded|prepared|skipped`；旧已评分项迁移为 `graded`。`submission_id` 有非空唯一索引，`result_comment` / `result_full_answer` / `result_answer_source` 保存首次点评、答案及来源，不保存用户回答正文。来源由后端决定：`stored`（题库）、`model`（模型）、`NULL`（历史记录或无答案自评）。重放不重新读取题库答案替换历史结果。

`init_db` 负责事务迁移，当前 `PRAGMA user_version = 3`，拒绝更高版本。v2→v3 保留旧题 ID/进度、会话、评分及答案快照，补齐题包/专题、有序会话和完成类型结构；多条旧 open 仍只保留最新一条且不修改题目调度。正式升级真实库前须另行备份完整 SQLite；v3 不能直接交给旧版程序使用，`.bagu-backup` 不是完整库备份。

`session_id`：`s_YYYYMMDD_` + 8 位 hex。用 `new_session_id()`，不要手写。

## 调度

`GRADE_INTERVALS`：again 走特殊分支（level=0，间隔 1 天）；hard/good/easy 为 1/3/7 天再乘 `LEVEL_MULT`（level 1–3 → 1/2/4，新升到的 level）。不要在未改 spec 的情况下改间隔。

抽题 SQL：到期复习优先，再随机新题。分类过滤用 `--cat` / `body.cat`。

## HTTP（当前）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InGnIJM/AI-Bagu](https://github.com/InGnIJM/AI-Bagu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
