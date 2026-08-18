---
trigger: always_on
description: 基于 Node.js 的 VRChat 好友在线状态监控服务，配合 Hermes 插件和桌面插件使用，支持自动抓取 OTP 验证码登录。
---

# VRChat 助手 (vrchat-assistant)

基于 Node.js 的 VRChat 好友在线状态监控服务，配合 Hermes 插件和桌面插件使用，支持自动抓取 OTP 验证码登录。

> 🤖 **AI-first 项目**：本程序只面向 AI Agent 使用与扩展，人类不直接编码；加功能 = 人类向 Agent 提需求 → Agent 实现。开发规范见 [DEVELOPMENT.md](./DEVELOPMENT.md)。

**面向 AI Agent 的配置引导**：以下步骤可由 Agent 自动完成。建议 agent 在本仓库目录内运行（服务目录可自动探测，无需设置环境变量）。

## 安装前检查（Agent 必读）

> ⚠️ **在开始配置步骤 1 之前，Agent 必须先向用户核实以下问题：**

1. **是否安装/使用过 VRCX？**
   - VRCX 是一款流行的 VRChat 第三方工具，会在本地保存好友活动历史、世界缓存、好友备注等数据（SQLite 数据库）。本服务的 `migrate-vrcx0.mjs` 脚本可从 VRCX 数据库一键导入这些历史数据。
   - 若用户**使用过 VRCX**，继续追问第 2 个问题。
   - 若用户**从未使用过 VRCX**，可跳过数据导入，直接进入「配置步骤」。

2. **是否需要将 VRCX 中的历史数据导入到本系统？**
   - 导入内容包括：好友列表与信任等级、好友备注/昵称、世界缓存、位置变更历史、上下线记录、Avatar 变更、状态/Bio 变更等（详见下表）。
   - 导入是**可选**的——不导入也能正常使用本服务，只是查询历史活动记录时没有迁移前的旧数据。
   - 若用户**需要导入**，在完成「配置步骤」1-3（服务能正常启动）后，运行 `node migrate-vrcx0.mjs` 导入数据（脚本会自动探测 VRCX 数据库路径和 userId）。
   - **迁移引擎（v1.1.0 起）**：改用 **better-sqlite3**（与主服务同引擎，WAL 模式），已移除旧版 sql.js 整文件重写（该方式曾导致 `SQLITE_CORRUPT`，2026-08-12 实测踩坑后由 PR #10 防呆 + PR #12 根因修复）。服务运行中迁移不再损坏数据库，但仍**建议迁移前停止服务**（避免与服务的实时写入交错）。脚本内置 127.0.0.1:8799 端口检测：检测到服务运行时会给出警告，需加 `--force` 确认后继续（风险自负）。
   - **✅ 幂等说明（v1.2.0 起，PR #14 已修复）**：迁移记录带 `vrcxId` 标记 + events 表 JSON 表达式唯一索引 + `INSERT OR IGNORE`，**重复执行自动跳过已迁移记录**（只补新增，不重复插入）。旧版脚本（无 vrcxId）产生的迁移数据会被检测并提示，需 `--force` 才会重插（防误全量重插）。迁移完成后 `node start-monitor.js` 启动服务。
   - 若用户**不需要导入**，直接进入「配置步骤」。

3. **是否愿意参与本仓库的 PR/issue 协作审核（AGENT-REVIEW 协议）？**
   - 本仓库支持**协作审核**：多个独立部署的 AI Agent 可以共同参与 PR / issue 审核，通过认领评论声明参与，满员（默认 3 人）后不再重复劳动。协议见仓库根目录 `AGENT-REVIEW.md`。
   - 参与是**完全自愿、可随时退出**的：不参与不影响任何功能；参与后 Agent 会定时检查 open PR/issue，认领未满员的条目并提交审核意见（只读，不合并）。
   - 若用户**愿意参与**：Agent 在完成「配置步骤」后，按 `AGENT-REVIEW.md` §1 配置定时检查（参考脚本 `scripts/agent-review.py`）。
   - 若用户**不愿意参与**：跳过即可，Agent 不得自行参与。

| 迁移项 | VRCX 数据表 | 说明 |
|--------|------------|------|
| 好友列表 | `friend_log_current` | 好友 userId、显示名、信任等级 |
| 好友备注 | `memos` | 好友昵称/备注文本 |
| 世界缓存 | `cache_world` | 世界 ID、名称、作者、缩略图等 |
| 位置变更 | `feed_gps` | 好友何时在哪个世界 |
| 上下线记录 | `feed_online_offline` | 好友上线/下线时间及位置 |
| Avatar 变更 | `feed_avatar` | 好友更换 Avatar 记录 |
| 状态变更 | `feed_status` | 好友状态文本变更 |
| Bio 变更 | `feed_bio` | 好友个人简介变更 |

> **数据库位置**：迁移脚本会按平台自动探测以下路径：
> -  **Windows**：`%USERPROFILE%\AppData\Roaming\VRCX\VRCX.sqlite3`（VRCX-0：`%USERPROFILE%\AppData\Roaming\VRCX-0\VRCX-0.sqlite3`）
> -  **Linux**：`~/.config/VRCX/VRCX.sqlite3`（原生 Electron 版）；若通过 Wine 运行 Windows 版：`~/.wine/drive_c/users/<user>/AppData/Roaming/VRCX/VRCX.sqlite3`（自定义 Wine 前缀可用 `WINEPREFIX` 环境变量指定）
> -  **macOS**：`~/Library/Application Support/VRCX/VRCX.sqlite3`
>
> 若自动探测失败，Agent 可让用户提供数据库路径手动指定：`node migrate-vrcx0.mjs <VRCX数据库路径> <userId>`。

## 获取代码（Fork & Clone）

> 💡 本项目是 AI-first 项目，用户会持续向 Agent 提需求、由 Agent 实现功能。为保留这些变更并便于同步上游更新，**建议 Agent 先 Fork 本仓库到用户自己的 GitHub 账号，再克隆 Fork 到本地**，而非直接 clone 原仓库。

- **推荐**：Fork 后克隆
  ```bash
  # Agent 通过 GitHub 工具完成 fork，然后：
  git clone https://github.com/<用户名>/vrchat-assistant.git
  cd vrchat-assistant
  # upstream 指向原仓库（fork 来源）
  git remote add upstream https://github.com/ggg123124/vrchat-assistant.git
  ```
  之后 Agent 实现的新功能可直接 `git push origin` 保存到用户的 Fork；需要同步官方更新时执行 `git pull upstream main`。

- **可选**：仅直接 clone 原仓库（适用于不打算修改代码、只用现成功能的用户）。注意此方式下 Agent 产生的代码变更无法推送到远程，仅保留在本地，存在丢失风险。

## 配置步骤

### 1. 配置凭据文件

复制 `credentials.example.json` 为 `credentials.json`，填入以下信息：

```json
{
  "email": "你的 VRChat 登录邮箱",
  "password": "你的 VRChat 登录密码",
  "imap_auth_code": "你的邮箱 IMAP 授权码"
}
```

> 注意：支持任意提供 IMAP 服务的邮箱（QQ/163/Gmail/Outlook 等），服务根据邮箱域名自动选择 IMAP 服务器。若需手动指定服务器，可在 `credentials.json` 中添加 `imap_host` 字段。

**获取邮箱 IMAP 授权码：**
各邮箱服务商的 IMAP 开启方式不同，通用步骤为：登录邮箱网页版 -> 设置 -> 开启 IMAP/SMTP 服务 -> 生成授权码/专用密码。以 QQ 邮箱为例：设置 -> 账户 -> POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV 服务 -> 开启 IMAP/SMTP 服务，按提示发送短信后生成授权码。

> `credentials.json` 已被 .gitignore 排除，不会提交到仓库。

### 2. 设置环境变量（可选）

- `VRC_MONITOR_DIR`：指向本仓库目录（克隆后服务所在目录）。若 agent 在仓库目录内运行，服务可自动探测，无需手动设置。
- `VRC_MONITOR_NODE`：指向 Node.js 可执行文件路径。若不设置，自动从 PATH 查找 `node`。
- `VRC_MONITOR_DB_PATH`：SQLite 数据库文件路径（默认 `<仓库>/vrc-monitor.sqlite3`）。可将数据库迁移到任意位置（如独立数据盘），配合常驻服务使用。
- `VRC_MONITOR_BACKUP_DIR`：自动备份目录（默认 `<仓库>/backups`）。
- `VRC_MONITOR_LOG_DIR`：常驻服务脚本的日志 / 修复记录目录（默认 `<仓库>/service-logs`，仅 `service-windows/` 脚本使用）。
- `VRC_MONITOR_PYTHON`：执行 fetch-otp.py 的 Python 解释器路径（默认 PATH 中的 `python`）。以计划任务 / systemd / 容器等方式运行且 PATH 中无 python 时必须设置，否则 OTP 自动登录失败会陷入重试循环（每次循环 VRChat 都会重新发送验证码邮件）。**路径含空格无需自带引号**（如 `C:\Program Files\Python311\python.exe`），脚本执行时会自动加引号。

### 3. 启动服务

```bash
node start-monitor.js
```

服务启动后自动完成：加载凭据 -> 校验 cookie -> 过期则自动从邮箱 IMAP 抓取 OTP 验证码登录 -> 建立 WebSocket 连接。

健康检查：

```bash
curl http://127.0.0.1:8799/health
```

**验证成功的标准**：返回 JSON 中 `auth.authenticated` 为 `true`、`ws.status` 为 `connected`、`friendState.online` 为在线好友数。

### 4. 安装 Hermes 插件（进程托管）

```bash
# 复制整个插件目录（含 dashboard 后端子目录，必须带 -r）
# <hermes home> 默认位置：Linux/macOS 为 ~/.hermes，Windows 为 %LOCALAPPDATA%\hermes
mkdir -p "$HERMES_HOME/plugins/vrc-monitor"
cp -r hermes-plugin/* "$HERMES_HOME/plugins/vrc-monitor/"

# 启用
hermes plugins enable vrc-monitor
```

插件提供 `vrc_status` / `vrc_start` / `vrc_stop` / `vrc_restart` 工具，并在每次 Hermes 会话开始时自动拉起服务（on_session_start 钩子）。

> **平台限制**：插件当前仅支持 Windows（`plugin.yaml` 中 `platforms: [windows]`）。macOS / Linux 用户需手动执行 `node start-monitor.js` 启动服务，或等待插件跨平台支持。Node.js 服务本身是跨平台的，仅 Hermes 插件托管层有此限制。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ggg123124/vrchat-assistant](https://github.com/ggg123124/vrchat-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
