---
trigger: always_on
description: > 项目级别速查手册。给后续会话（人或 AI）秒接上下文用。任何会改契约的提交请同步更新本文档。
---

# AGENT.md — AI Work 助手 (ai-work-assistant) v3.6.1

> 项目级别速查手册。给后续会话（人或 AI）秒接上下文用。任何会改契约的提交请同步更新本文档。
> 注：品牌已由 Trae Work Assistant 迁移为 **AI Work 助手（ai-work-assistant）**，本机仓库目录暂为 `trae-work-assistant`，后续可整体重命名。

## 1. 一句话

Windows 桌面端多账号签到 + 登录态切换 + 设备隔离 + API 网关一站式工作台，**深度支持 Trae Work 与 Trae（Trae CN IDE）双应用**（账号自动发现、切换/快照按目标应用独立、账号池 app 无关同池调度；桥档案表已预留豆包 / WorkBuddy）。**所有数据仅存在 `%APPDATA%\AIWorkAssistant\`，零外部网络**。

## 2. Quick Start

```powershell
# 仅 Windows，需要 Node 18+ / Rust stable (MSVC) / VS Build Tools C++ 工作负载 / WebView2（Python 已移除，scripts 全 .mjs 零依赖）
cd ai-work-assistant   # 本机目录暂为 trae-work-assistant，见文首说明
npm install
npm run tauri dev          # 开发模式（Tauri WebView 加载 Vite 5173）
npm run tauri build        # 打包 MSI + NSIS 到 src-tauri/target/release/bundle/
node scripts/rename_release.mjs     # 产物统一输出到 release/，中文命名 AI Work 助手_<版本>_x64*
```

测试：

```powershell
cargo test                                    # Rust 单测（需先装工具链；切换器/签到/网关全覆盖）
```

## 3. 技术栈

| 层 | 技术 |
|---|---|
| 外壳 | Tauri 2.x (Rust 1.75+ MSVC) |
| 前端 | React 18 + TypeScript 5 + Vite 5 + Tailwind 3 + Zustand 4 + Recharts 2 + lucide-react |
| 后端 | Rust (serde / chrono / axum / ureq / tauri-plugin-{shell,dialog,notification,single-instance}) |
| 辅助 | Node.js 18+（scripts/*.mjs 发布工具链，零 npm 依赖）；PowerShell 运行时依赖已移除（switcher 模块进程内直调） |

## 4. 目录地图

```
ai-work-assistant/
├── AGENT.md                      # 本文件（项目速查）
├── README.md                     # 用户文档
├── package.json / vite.config.ts / tsconfig.json / tailwind.config.js / postcss.config.js / index.html
├── docs/                         # user-manual / tech-framework / product-design / backlog（唯一待办；原五份分析文档 2026-09-13 归并删除，原文在 git 历史）
├── src/                          # 前端
│   ├── App.tsx                   # 外壳（TitleBar + Sidebar + TopBar + 页面切换 + Toaster）
│   ├── store.ts                  # Zustand 单一真相（init / 刷新 / checkin/switch/saveLogin 事件归约）
│   ├── types.ts                  # 与 Rust DTO 对齐（snake_case）
│   ├── lib/                      # tauri.ts(invoke 封装+事件订阅) / themes.ts(主题) / delay.ts(withMinDelay) / cn.ts / about.ts / useIsDark.ts
│   ├── components/               # TitleBar/Sidebar/TopBar/Toaster/PageHeader/SetupGuide/ui + SystemDialog(系统设置+系统日志弹框)/GeneralSettingsPanel/AboutDialog
│   └── pages/                    # Dashboard / Accounts(661行编排 + accounts/ 16 个拆分子组件) / Checkin / Credits / Logs / ApiService / Settings
├── scripts/                      # dev-tauri.mjs(tauri 脚本入口) / sync_version.mjs / rename_release.mjs / package_portable.mjs / gen_asset_base64.mjs
├── src-tauri/
│   ├── tauri.conf.json           # 无装饰窗 / 无外部资源（Python 与 PS 桥均已移除，全 Rust）
│   └── src/
│       ├── main.rs               # 注册全部命令 + --task-run CLI 任务模式
│       ├── state.rs              # AppState（%APPDATA%\AIWorkAssistant + 旧目录迁移）
│       ├── models.rs             # DTO（含 CheckinSummary.time 字段）
│       ├── store/                # SQLite 存储层（v3.4.5 起全量承载 data 目录状态，替代 JSON 文件读写）
│       │   ├── mod.rs / schema.rs / migrate.rs  # 连接注册表(WAL) / 建表 / 启动迁移(旧 JSON 导 backup/)
│       │   └── *.rs              # kv 键值文档表 + 行文档实体表 + 列化流水表类型化读写
│       ├── device_proxy/         # MITM 代理模块（hyper+rustls 自建，原 Python 迁移）：CA/头改写/凭据捕获/WS 帧记录/SSE 摘要/上游透传
│       ├── fs_utils.rs           # 原子 read_json / write_json（vault/日志/导出仍用）/ mask / 时间辅助
│       ├── vault.rs              # Stronghold 凭证保险库（DPAPI 主密码；JSON 落盘占位化）
│       ├── workbuddy_cli.rs      # CLI 切号桥决策（decide_target 纯函数 + 轮换线程）
│       ├── jwt.rs                # parse() + status_of() + refresh() + oauth_parse()
│       ├── api_server/           # API 网关模块
│       │   ├── mod.rs            # 常量 + 路由注册
│       │   ├── server.rs         # axum 服务器启停
│       │   ├── routes.rs         # OpenAI /v1/chat/completions + Anthropic /v1/messages + Codex /v1/responses（SSE 流式 + 非流式，三协议输出）
│       │   ├── pool.rs           # 账号池调度（积分感知 + 冷却状态机 + 账号轮换，app 无关）
│       │   ├── payload.rs        # OpenAI/Anthropic 请求 → llm_utils_chat 改写（anthropic_to_openai 先转内部格式）
│       │   ├── sse.rs            # SSE 协议转换（SOLO → OpenAI chunk / Anthropic 事件流）
│       │   ├── auth.rs           # API Key 鉴权（Bearer + x-api-key 双风格）
│       │   ├── models_sync.rs    # 模型列表配置化（api_models.json）+ 官网 batch_get_detail_param 同步
│       │   └── api_logger.rs     # API 请求日志
│       └── commands/             # env / cert / proxy / accounts / checkin / switch / misc / profile / api_server / oauth / trae_apps(双应用发现) / process(三级关闭) / updater / wb_config(wb 手工配置读写)；workbuddy/ 为目录模块（common/accounts/checkin/credits/cli/chatdata/oauth/env_reset，mod.rs pub use 保持命令路径不变）
│       └── tasks/                # 后台业务直调模块（trae_checkin / wb_checkin / wb_common / wb_credits / doubao_session / doubao_quota / doubao_chats / ui_click）
│           ├── trae_checkin.rs   # 批量签到（vault 解密内存传递，无子进程）
│           ├── wb_checkin.rs     # WorkBuddy 签到/成长/续期（run_checkin_round / run_growth_round / run_renew_only）
│           └── doubao_*.rs       # 豆包会话续期 / 额度巡检 / 对话导出
│       └── switcher/            # 登录态切换器（原 PS 桥 trae-switch-bridge.ps1 全量 Rust 化）
│           ├── mod.rs           # run_action 入口 + 7 Action 编排 + ProgressSink（TauriSink/CliSink）
│           ├── profile.rs       # 5 应用 × 3 快照布局档案表（icube/chromium/authfile）
│           ├── locate.rs        # exe 六级发现（settings→候选→lnk→注册表→进程→缓存）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smart-open/TraeWorkAssistant](https://github.com/smart-open/TraeWorkAssistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
