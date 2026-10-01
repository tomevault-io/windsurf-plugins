---
trigger: always_on
description: > 项目级别速查手册。给后续会话（人或 AI）秒接上下文用。任何会改契约的提交请同步更新本文档。
---

# AGENT.md — Trae Work Assistant v2.4.7

> 项目级别速查手册。给后续会话（人或 AI）秒接上下文用。任何会改契约的提交请同步更新本文档。

## 1. 一句话

Windows 桌面端多账号签到 + 登录态切换 + 设备隔离 + API 网关工具。**所有数据仅存在 `%APPDATA%\TraeWorkAssistant\`，零外部网络**。

## 2. Quick Start

```powershell
# 仅 Windows，需要 Node 18+ / Rust stable (MSVC) / VS Build Tools C++ 工作负载 / WebView2
cd trae-work-assistant
npm install
npm run tauri dev          # 开发模式（Tauri WebView 加载 Vite 5173）
npm run tauri build        # 打包 MSI + NSIS 到 src-tauri/target/release/bundle/
```

测试：

```powershell
python src-python/tests/test_auto_checkin.py   # Python 纯函数单测
cargo test                                    # Rust 单测（需先装工具链）
```

## 3. 技术栈

| 层 | 技术 |
|---|---|
| 外壳 | Tauri 2.x (Rust 1.75+ MSVC) |
| 前端 | React 18 + TypeScript 5 + Vite 5 + Tailwind 3 + Zustand 4 + Recharts 2 + lucide-react |
| 后端 | Rust (serde / chrono / axum / ureq / tauri-plugin-{shell,dialog,notification}) |
| 辅助 | Python 3.9+（仅标准库 + `cryptography`）+ PowerShell 5.1+（系统自带） |

## 4. 目录地图

```
trae-work-assistant/
├── AGENT.md                      # 本文件（项目速查）
├── README.md                     # 用户文档
├── package.json / vite.config.ts / tsconfig.json / tailwind.config.js / postcss.config.js / index.html
├── docs/                         # 设计/技术/API/用户手册/operation-manual
├── src/                          # 前端
│   ├── App.tsx                   # 外壳（TitleBar + Sidebar + TopBar + 页面切换 + Toaster）
│   ├── store.ts                  # Zustand 单一真相（init / 刷新 / checkin/switch/saveLogin 事件归约）
│   ├── types.ts                  # 与 Rust DTO 对齐（snake_case）
│   ├── lib/tauri.ts              # invoke 封装 + 事件订阅（setupListeners）
│   ├── components/               # TitleBar/Sidebar/TopBar/Toaster/PageHeader/SetupGuide/ui
│   └── pages/                    # Dashboard / Accounts / Checkin / Credits / Logs / ApiService / Settings
├── src-tauri/
│   ├── tauri.conf.json           # 无装饰窗 / bundle.resources = ../resources/python/ + ../src-ps/ + ../resources/browser/
│   └── src/
│       ├── main.rs               # 注册全部命令
│       ├── state.rs              # AppState（%APPDATA%\TraeWorkAssistant + python_dir）
│       ├── models.rs             # DTO（含 CheckinSummary.time 字段）
│       ├── fs_utils.rs           # 原子 read_json / write_json / mask / 时间辅助
│       ├── jwt.rs                # parse() + status_of() + refresh() + oauth_parse()
│       ├── python.rs             # spawn_script（注入 TRAEDATA_DIR）
│       ├── api_server/           # API 网关模块
│       │   ├── mod.rs            # 常量 + 路由注册
│       │   ├── server.rs         # axum 服务器启停
│       │   ├── routes.rs         # OpenAI 兼容路由（SSE 流式 + 非流式）
│       │   ├── pool.rs           # 账号池调度（积分感知 + 冷却状态机 + 账号轮换）
│       │   ├── sse.rs            # SSE 协议转换（SOLO → OpenAI chunk）
│       │   ├── auth.rs           # Bearer Token 鉴权
│       │   └── api_logger.rs     # API 请求日志
│       └── commands/             # env / cert / accounts / checkin / proxy / switch / misc / profile / api_server / oauth
├── src-python/
│   ├── device_proxy.py           # MITM 代理（env TRAEDATA_DIR、--gen-ca）
│   ├── auto_checkin.py           # 批量签到（--json-stream / --accounts / --scope）
│   ├── requirements.txt          # cryptography
│   └── tests/test_auto_checkin.py
└── src-ps/trae-switch-bridge.ps1 # 非交互切换桥 + NDJSON 步骤输出
```

## 5. Tauri 命令契约

> **调用约定**：invoke 的**顶层参数名**跟随 Rust 函数签名（驼峰不替换，参数名直接匹配）。**嵌套对象**（`opts` / `patch`）的字段名保持 **snake_case**（Tauri 默认 serde 字段名，不做 camelCase 转换）。

| 模块 | 命令 | 说明 |
|---|---|---|
| 环境 | `env_check` → `EnvStatus` | `installed/running/version/path` |
| 证书 | `cert_status` / `cert_install` | 安装走 UAC `certutil -addstore -f Root` |
| 代理 | `proxy_start(port)` / `proxy_stop()` / `proxy_status()` | ProxyStatus：`running/port/captured/started_at` |
| 账号 | `accounts_list` → `AccountView[]` | 聚合 JWT / 分组 / 设备 / 积分 / 今日 |
| 账号 | `account_add_manual(name, jwt, groupId?)` | 解析 JWT → userId 入库 |
| 账号 | `account_delete(userId, deleteProfile)` | 同时清分组；`deleteProfile=true` 删 profiles/<uid> |
| 账号 | `account_oauth_add` → 实际为 `oauth_login(callback_url, account_name?, group_id?)` | 从 OAuth 回调 URL 解析 token + userInfo |
| OAuth | `oauth_get_login_url()` → `{ url }` | 构造 Trae 登录 URL |
| OAuth | `oauth_parse_callback(callback_url)` → `{ user_id, ... }` | 解析回调 URL 中的 token |
| 分组 | `groups_list` / `group_create` / `group_update` / `group_delete` / `group_move` | 删除分组时账号回落「未分组」 |
| 签到 | `checkin_start(opts)` → NDJSON 事件 | `opts: { scope, user_ids?, skip_checked_in, skip_expired }` |
| 切换 | `switch_account(userId)` | 调 `trae-switch-bridge.ps1 -Action Switch` |
| 保存 | `save_current_login(userId)` | 调 `trae-switch-bridge.ps1 -Action SaveCurrentLogin` |
| 快照 | `profile_list` → `ProfileInfo[]` | 列出 data/profiles/ 下所有快照 |
| 快照 | `profile_backup(userId)` / `profile_restore(userId)` / `profile_delete(slot)` | 手动备份/恢复/删除 |
| 设备 | `device_reset(userId)` | 删 `device_map.json[ uid ]` |
| JWT | `jwt_parse(jwt)` / `refresh_jwt(userId)` | 解析 / 自动刷新（需 refresh_token） |
| API | `api_server_start(port)` / `api_server_stop()` / `api_server_status()` | API 网关启停 |
| API | `pool_list` / `pool_set` / `pool_status` | 账号池管理 |
| API | `api_debug_toggle` / `api_debug_status` | API 请求日志开关 |
| 日志 | `logs_query({ opts: { log_type, date, keyword, limit } })` → `LogLine[]` | `split_time` 会 strip BOM 前缀 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cxqc168-wq/Trae-workbuddyAssistant](https://github.com/cxqc168-wq/Trae-workbuddyAssistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
