---
trigger: always_on
description: >-
---


# Scholar Kit

学术文献检索与科研辅助工具。脚本是"手"，Agent 是"脑"。

## Quick Start

```bash
# 0. 安装依赖（首次）
<python> -m pip install -r <skill_path>/scripts/requirements.txt

# 1. 搜索（用户要求核心期刊时加 --core）
<python> <skill_path>/scripts/literature.py search "乡村振兴" --core 北大核心,CSSCI

# 2. 获取全文（指定序号或前 N 篇）
<python> <skill_path>/scripts/literature.py read-detail --indices 1,3,9 --fulltext

# 3. 导出引用
<python> <skill_path>/scripts/literature.py cite --style gbt7714
```

> `<skill_path>` 是本 Skill 目录的实际路径，Agent 应根据自身环境自动解析；`<python>` 是按 [Python 解释器发现](#python-解释器发现) 解析出的命令。

## 何时使用 / 不使用

**使用**：用户要搜论文、下论文、写综述、加引用、选题分析、格式化参考文献、优化论文表达、引文追踪、研究趋势分析、文献对比、阅读笔记
**不使用**：通用网页搜索、非学术内容、代码文档查找、翻译（无文献检索需求时）

## ⚠ 沙盒环境（Codex / Cursor / Claude Code / Gemini CLI / 本地 Agent）

知网功能需要 Selenium 浏览器自动化 → 需要 msedgedriver → 首次运行需联网下载驱动（~8MB）。沙盒默认可能禁网，但**所有主流平台都支持运行时提权**。

**Agent 遇到 `sandbox_blocked` 或驱动缺失时，必须以提权方式重试，不得直接放弃或回退 API 源。** 提权后平台会弹窗让用户授权。

当 `check.capabilities.needs_escalation: true` 时，Agent 必须把 `agent_action` 和 `retry_command` 视为下一步指令：携带 `recommended_env` 重新以提权权限运行同一检查或知网命令。未完成一次提权重试前，不得把 CNKI 判定为不可用，也不得直接回退到 API 源。

各平台提权方式：

| 平台 | 提权方法 |
|------|---------|
| **Codex** | 在 `~/.codex/config.toml` 或项目本地未跟踪的 `.codex/config.toml` 中配置 `network_access = true`；也可运行 `check --fix` 生成本地配置。Codex 会在联网时弹窗让用户确认；不要将 `.codex/` 提交到仓库 |
| **Cursor** | Shell 工具加 `required_permissions: ["all"]` |
| **Claude Code（交互式）** | 用 `/permissions`、`permissions.allow/ask/deny` 或合适的 permission mode 批准 Bash/PowerShell、WebFetch、浏览器/网络相关命令；启用 Bash sandbox 时可用 `/sandbox` 配置允许域名、路径和是否允许失败后转普通权限流 |
| **Claude Code（子 agent/SDK）** | Bash/PowerShell 工具调用时添加 `dangerouslyDisableSandbox: true`（需要网络的命令：`check`、`search --source cnki`、`batch-search`、`download`、`batch-download`、`read-detail`）。**遇到 Exit code 49 表示权限被拒绝，必须添加此参数** |
| **Gemini CLI** | 若启用 `--sandbox` / `-s`、`GEMINI_SANDBOX` 或 `settings.json` 的 sandbox 配置，遇到网络/路径限制时批准 Sandbox Expansion Request，或调整 sandbox profile、`SANDBOX_MOUNTS`、代理、`security.toolSandboxing` 后重启 Gemini CLI；浏览器自动化在容器/系统沙箱中可能不可用，需改用有 GUI 和网络权限的本机会话 |
| **其他本地 Agent** | 按该平台的 allowlist、approval、sandbox expansion、unsandboxed retry 或提权参数重跑 `retry_command`，并携带 `recommended_env`；没有提权机制时，明确要求用户在有网络和浏览器权限的本地终端执行，不要静默回退 API 源 |

Codex 本地配置示例（不要提交到仓库）：
```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = true
```

`check --fix` 会自动将此配置写入项目本地 `.codex/config.toml` 和用户级 `~/.codex/config.toml`；`.codex/` 应保持未跟踪。

仅使用 API 源（OpenAlex/S2/arXiv/NSSD/DBLP/BASE）时不需要提权，直接 `--source openalex` 即可。

## 前置条件

**运行环境**: Python 3.9+, Selenium 4.10+, Edge 或 Chrome, 知网需校园网/VPN。

### Python 解释器发现

Agent 不要假设 `python` 一定在 PATH 中。首次调用脚本前，应先解析可用 Python 命令，并在同一会话后续命令中复用：

1. 若环境变量 `PYTHON` 存在，优先使用 `$PYTHON`
2. **Windows 优先尝试 `py -3`**（Python Launcher，最可靠）
3. 否则尝试 `python3`
4. 否则尝试 `python`
5. 全部不可用时，才提示用户安装 Python 3.9+ 或将 Python 加入 PATH

**遇到 Exit code 127（command not found）表示 Python 命令不存在，必须按上述顺序重新解析。**

验证命令示例：

```bash
if [ -n "$PYTHON" ]; then
  "$PYTHON" --version
elif command -v py >/dev/null 2>&1; then
  py -3 --version
elif command -v python3 >/dev/null 2>&1; then
  python3 --version
elif command -v python >/dev/null 2>&1; then
  python --version
else
  echo "Python 3.9+ not found"
fi
```

下文命令中的 `python` 代表上述已解析出的解释器命令，不是固定字符串。

Agent 在首次调用脚本前应运行 `check` 命令自检（同一会话只需运行一次，Agent 应缓存 `capabilities` 结果供后续命令使用）：

```bash
python scripts/literature.py check --fix
```

`--fix` 自动安装缺失的 selenium、检测沙箱环境并写入网络配置、重试知网连通性。Agent **不得因为 `check.status == "warning"` 或某个可选项 `fail` 直接退出**；必须读取 `capabilities` 决定下一步。只有 Python 无法运行、Python < 3.8、或当前任务必需能力不可用且无替代路径时，才算阻断。

`check` 返回的 `capabilities` 字段是 Agent 的决策依据：

- `cnki_feasible: true` → 可使用全部命令
- `cnki_feasible: false` → **以提权方式重试，不要放弃**：
  0. 若存在 `needs_escalation: true` → 按 `retry_command` 重新运行，并按 `recommended_env` 设置环境变量；`agent_action: rerun_same_command_with_escalated_permissions` 表示必须请求平台提权
  1. 设置 `SCHOLAR_SKIP_NETWORK_CHECK=1`，然后**以提权方式重新运行知网命令**（平台会弹窗让用户授权网络访问）。提权后 Selenium Manager 自动下载驱动，三个卡点一步清零
  2. 提权后仍失败 → 读 `fix_note`，**区分场景**：
     - 硕士/博士/学位论文、知网全文等**只有知网能做的任务** → 告知用户"请确认已连接校园网/VPN"，**不要用 API 源替代**
     - 其他通用搜索 → 用 `--source openalex` 继续，一句话告知用户

- `python-docx` / `openpyxl` 失败只影响 Word/Excel 功能，不影响搜索、引用、下载；需要时降级输出 Markdown/JSON。
- `selenium`、浏览器、驱动、知网连通性失败只影响 CNKI 自动化；若用户任务不是 CNKI 专属，可用 API 源继续。
- `api_sources: true` 时，OpenAlex/Semantic Scholar/arXiv/NSSD/DBLP/BASE 相关搜索不应因 CNKI 检查失败而中止。

- `update.update_available: true` → 提示用户"有新版本可用，在 skill 目录执行 `git pull` 更新"（该字段仅在版本检测成功时存在，缺失时忽略）

详见 [平台兼容性](references/environment.md#平台兼容性)。

### 配置

优先级: **环境变量 > `.scholar-kit/config.json` > 内置默认值**

| 配置项 | 环境变量 | config.json 键 | 默认值 |
|--------|----------|----------------|--------|
| 知网请求间隔 | `SCHOLAR_REQUEST_INTERVAL` | `request_interval` | `3` |
| 缓存 TTL（天） | `SCHOLAR_CACHE_TTL_DAYS` | `cache_ttl_days` | `30` |
| API 邮箱 | `SCHOLAR_MAILTO` | `mailto` | `scholarkit@example.com` |
| 下载目录 | `SCHOLAR_SAVE_DIR` | `save_dir` | `./papers` |
| 浏览器 | `SCHOLAR_BROWSER` | `browser` | `auto` |
| 批量下载窗口大小 | `SCHOLAR_BATCH_WINDOW_SIZE` | `batch_window_size` | `10` |
| 跳过网络预检 | `SCHOLAR_SKIP_NETWORK_CHECK` | — | `0`（沙盒中建议设为 `1`） |
| 浏览器驱动路径 | `SCHOLAR_DRIVER_PATH` | — | 自动（手动指定 msedgedriver/chromedriver 路径） |
| Selenium 缓存路径 | `SE_CACHE_PATH` | — | 自动（默认缓存不可写时降级到 `.scholar-kit/selenium-cache`） |

## Agent 与脚本的分工

| Agent 负责 | 脚本负责 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lottshin/scholar-kit](https://github.com/lottshin/scholar-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
