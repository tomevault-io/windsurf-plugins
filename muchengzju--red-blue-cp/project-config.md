---
trigger: always_on
description: > 本仓库的规则书，所有 AI 编码 agent（Claude Code / Codex / Cursor / Copilot / Gemini 等）在本仓库工作时读这份。
---

# AGENTS.md · Red Blue CP

> 本仓库的规则书，所有 AI 编码 agent（Claude Code / Codex / Cursor / Copilot / Gemini 等）在本仓库工作时读这份。
> `CLAUDE.md` 通过 `@AGENTS.md` 指向本文件（单一真相源），只在那边追加 Claude Code 专属内容。
> 修改前先看 `docs/PRD.md` / `docs/SPEC.md`，理解上下文。
>
> **仓库布局**：根目录只留 `README.md` / `AGENTS.md` / `CLAUDE.md` / `LICENSE` / `CHANGELOG.md`；其余规划/设计/发布文档（`PRD.md` / `SPEC.md` / `PLAN.md` / `DESIGN.md` / `REFERENCES.md` / `RELEASING.md` / `LOG.md`）全在 **`docs/`** 下。下文凡出现 `PLAN.md`、`SPEC.md` 等裸名，路径都是 `docs/` 里那一份。

## 项目目标（一句话）

**Red Blue CP（红蓝CP）—— 自古红蓝出 CP，B 站小红书 Content Pipeline。**

把 B 站和小红书的视频/图文内容转成纯文本，沉淀成本地 Markdown 知识库；并在原文之上生成"速览"（高亮/卡片/脉络）。

## 当前阶段

**0.6.0 已发布**（PyPI + GitHub Release，MIT 开源）：速览产品 Pipeline = **Extract→Digest→Render**。`service/`→`extract/`（采集转录→忠实原文，无损）+ 新 `digest/`（原文→高亮/卡片/脉络，有损 LLM，确定性服务端锚定，与 extract 隔离）+ `app/config.py`（platformdirs 配置发现）+ **CLI** `rbcp digest/ls` + **`desktop/`**（RBCP Desktop = Tauri v2 + PyInstaller sidecar，内置 serve，全功能：任务/文件库/账单/设置/速览阅读器）。612 测试。桌面端本版仅 **macOS arm64、未签名**（首开右键→打开过 Gatekeeper），引擎/CLI/WebUI 全平台走 PyPI。
**历史**：0.5.2 = P0 URL→Markdown 闭环、博主全量/评论、M4 安全批量、M5a 流式+用量、M5b WebUI v2。**浏览器插件 0.3.1**（油猴主推）。

**桌面端注意**：跑的是**冻结的 PyInstaller sidecar**（`desktop/src-tauri/binaries/rbcp-serve-<triple>`）。改了 Python 源码必须 `desktop/sidecar/build.sh` 重打 + 重启 app 才生效；前端（`desktop/frontend/`）改动靠重载（Cmd+R）。

**下一步**：托管版 RBCP Cloud（按实际使用计费）、手机端、桌面端跨平台（Windows）——走私有云，引擎层继续开源（串行往后，未设计透）。

调整阶段/范围改 PLAN.md，加功能改 PRD.md，改实现改 SPEC.md——**改文档前对话里说一句**。

## 构建 / 测试 / 验证（命令可直接复制）

```bash
uv sync                                   # 建环境
./.venv/bin/pytest -q tests               # 跑测试（当前 612）
./.venv/bin/python -m compileall app/     # 至少能编译
bash scripts/check-leaks.sh               # 提交前必跑：查个人/环境信息泄漏
uv run rbcp serve                         # 起本地服务（默认 :8000）
uv run rbcp digest "<链接>"               # 转录 + 生成速览
cd desktop/sidecar && bash build.sh       # 桌面端 sidecar 重打（改 Python 源码后必须）
```

前端是纯 JS（`desktop/frontend/`、`app/web/templates/`），无构建步骤；改完用 `node --check <file>` 过语法。发布流程见 [RELEASING.md](docs/RELEASING.md)。

## 不变量（红线，违反就是 bug）

### 安全

1. **文件下载/读取接口必须走 `job_id`**，不允许用户传任意 file path。任何 `GET /api/files/{path}` 这类设计都是路径穿越漏洞，禁止。
2. **敏感配置不进 Git**。百炼 API Key 和小红书 cookie 存放在 `.env` 文件中，必须在 `.gitignore` 里。

### 部署

3. **MVP 仅支持单进程 uvicorn**。启动命令禁止 `--workers > 1`。asyncio.Queue / create_task 是进程内的，多 worker 会让任务状态混乱。
4. **不允许把数据库或日志写到 `~/transcript/`**。知识库目录只放 Markdown 文件。SQLite 索引文件 `_index.sqlite` 是唯一例外。

### 持久化

5. **媒体文件不进知识库**。知识库目录（`~/transcript`）只放 Markdown + `_index.sqlite`。默认媒体（音频流、图片）只存在于 `tempfile.TemporaryDirectory`，任务结束自动清理（`with` 块保证）。**P1 例外**：用户开 `--save-media` 时，原始媒体存到独立的 `RBCP_MEDIA_DIR`（默认 `~/transcript-media/`），仍**不混入知识库目录**。缩略图等派生媒体存 platformdirs 缓存目录，同样不进知识库。
6. **失败任务必须留痕**。SQLite 里有 status=failed + error_message + log_excerpt。WebUI 必须能展示。
7. **Markdown 写入必须原子**。先写 `{path}.tmp`，再 `os.replace` 替换。中途崩溃不能留半个文件。

### 业务

8. ~~不删 MCP 入口~~（**已废除**）。P0 采用参考移植方案，不 fork 上游，无 MCP 入口。如未来需要 MCP 能力，作为 P2 新建。
9. **不引入 bilibili-cli / xiaohongshu-cli**。P0 不依赖。博主全量/评论的**抓清单**：**浏览器插件（MV3 拦 `user_posted`）为主路径**（见 [博主安全批量](docs/blogger-safe-batch-feature.md)）；**pydoll 拦截器降为不稳定、不保证维护的可选项**（M2b 已交付的 pydoll 版保留不删，留给无浏览器/脚本化场景）。仍不用这两个 CLI。
10. **不自动判断 B 站字幕质量**。字幕优先是默认行为，"切 ASR" 是 P1 的手动按钮。不要写"如果字幕长度小于 X 就走 ASR"这种启发式。
11. **小红书图文图片处理走双轨**：URL 优先喂 VLM，失败回退到 tempfile 下载（保留 `referer` 等 headers），喂完即删。不要把 URL 当唯一稳定路径。

### 范围

12. **不做抖音**。即便上游仓库支持，本项目范围明确不包含。
13. **不做飞书集成、不做 PaddleOCR、不做 FTS5、不做 SSE**。这些是 P2，P0 一律不写代码也不留半成品入口。

## 工程纪律

### 反过度抽象（原则长期有效）

> 0.6 起代码组织：`extract/`（引擎，原 service/）+ `digest/`（0.6 新增，有损 LLM 层，与 extract 隔离）+ `web/` + `cli.py` + `desktop/`（Tauri 壳）+ `app/config.py`。`app/service/` 是弃用 re-export shim（0.7 删）。下面的反过度抽象原则不变。

**不要引入**以下抽象（即使你觉得"以后会用到"）：
- `PlatformAdapter` / `BiliAdapter` / `XhsAdapter` 类
- `Pipeline` 接口和 `BiliVideoPipeline` 等实现
- `JobQueue` 类（P0 用 `asyncio.create_task`，P1 才引入 `asyncio.Queue`）

**P0 允许的抽象**：`ModelProvider` Protocol + `DashscopeProvider`（参考移植的初始设计，不是提前抽象）。

P0 内部分发用 `if/elif platform == 'bilibili':` 就够了。看起来"丑"但这是 P0 应该的样子。

### 文件名 sanitize 必须严格

按 SPEC §6.2 实现。测试用例至少覆盖：
- 标题含 `/` `\` `:` `*` `?` `"` `<` `>` `|`
- 标题含 emoji
- 标题超过 60 字符
- 作者名为空 → `unknown_author`
- 标题为空 → 用 `note_id` 当文件名
- 同标题重复 → suffix 拼 BV_id / note_id 避免冲突

### 测试

P0 不要求高覆盖率，但**必须有以下集成测试**：
- 三种内容类型各一条 happy path 链接
- 一条故意失败的链接（验证 failed 持久化）
- 路径穿越测试（验证 API 拒绝 `../` 路径）
- tempfile 清理测试（任务后 tempfile 目录为空）

### 提交粒度

每完成 PLAN.md 里一个子任务（如 "extract/extractor.py" 或 "digest/anchor.py"），独立 commit。commit message 用现在时祈使句，参照 conventional commits：
```
feat(extractor): 实现 extract_url 包装上游 extract 函数
fix(markdown): 修复 emoji 标题导致的 sanitize 报错
```
**不自动 push**，等人说。合并走 PR（见 [RELEASING.md](docs/RELEASING.md)）。

### 多 session 并行协作（多开 agent 同时做不同特性）

同时开多个 agent 改这个仓库时，**必须用 git worktree，禁止同一工作目录开多个会话**（同目录多会话改同一批文件必撞）。Claude Code 原生支持：`git fetch origin` 后 `claude -w <name> -n <显示名>` 给会话开独立 worktree + 新分支；其它 agent 手动 `git worktree add`。

规则：

1. **各流各分支，从最新 main 切**：开 worktree 前先 `git fetch origin`，否则从过时基线切，看不到最新代码/契约（已踩过，见 [并行判断+调研分层](docs/devlog/2026-06-03-when-to-parallelize-and-research-layering.md)）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MuChengZJU/red-blue-cp](https://github.com/MuChengZJU/red-blue-cp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
