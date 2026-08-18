---
trigger: always_on
description: [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)（dsh，DeepSeek agent harness CLI）的 Windows 桌面壳应用：Tauri 2 窗口内嵌 dsh 官方 Web UI，Node.js 与 dsh 随安装包分发、装完即用。
---

# DSHDesktop

[deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)（dsh，DeepSeek agent harness CLI）的 Windows 桌面壳应用：Tauri 2 窗口内嵌 dsh 官方 Web UI，Node.js 与 dsh 随安装包分发、装完即用。

## 技术栈与形态

- **Tauri 2 + Rust**（`src-tauri/`）：进程监督、运行时部署、托盘、通知、主题跟随、诊断命令
- **Svelte 5 + TypeScript**（`src/`）：启动画面（splash）、诊断面板、其它设置三个本地页面；主界面是导航到的远程 dsh Web UI（`http://127.0.0.1:<port>`）
- 安装包：NSIS（`pnpm tauri build`），单实例、托盘常驻、关窗默认隐藏到托盘（可在"其它设置"改为直接退出）

## 目录结构

```
src-tauri/src/
  lib.rs            Builder 组装：插件(single_instance 必须最先；window-state 记忆窗口几何，
                    flags 不含 VISIBLE 防托盘隐藏态被记住；restore 在 window_created 排队、
                    早于首个可见帧落地，故主窗口 visible(false)+center() 创建
                    （首启居中，有记忆几何则被覆盖）、on_page_load(Finished) 再 show 不闪变)
                    → setup(代码创建主窗口，挂 on_download) → 事件桥(dsh-ready→导航)
  download.rs       主窗口下载处理：on_download 只能挂 builder（conf 窗口无法附加），
                    目标统一改到系统下载目录并 " (n)" 去重，完成/失败弹 toast + 记 events.log；
                    缺它时 wry 默认 handler 静默放行且抑制 WebView2 下载 UI，文件无声消失
  presets.rs        启动期修复 shipped minimal(极简模式)预设：rc 里它无条件挂载 PTY
                    持久 bash，而终端检查器未实现 win32 → 必抛 terminal inspection 错；
                    原地改写为 tool-pwsh 变体（persona 补 {{cwd}} 与 PowerShell 事实），
                    签名门控(上游加 win32 分支即停手)+marker 幂等，spawn 前完成；
                    只读判定 preset_signature_state 与契约套件共用
  upstream.rs       dsh 上游内部事实单一来源（入口/命令形/WS 端点与帧/设置键/
                    cordis patch/预设签名/改写 needle，每条注明出处与影响面）；
                    跟版红了只改这一个文件；tests/upstream_contract.rs 对真实运行时
                    逐项探测（无运行时自动 skip，CI 里 fetch-runtime 在 cargo test
                    之前故必真跑），DRIFT 输出直接指出改哪条常量、影响哪个模块
  platform/         平台抽象 trait（多平台预留）；windows.rs 实现（含 job 模块：
                    全局 KILL_ON_JOB_CLOSE Job Object，register_child 把每个子进程挂进去，
                    父进程被强杀时内核连带回收整树，防孤儿锁 runtime）；macos/linux 待实现
  process.rs        DshProcess 监督循环：spawn node bin.js web --port N、指数退避、stop/restart
  runtime.rs        ensure_runtime：安装目录可写则原地运行内嵌运行时；只读则回退部署副本
                    （.version 比对）；原地模式会清理旧版留下的 %LOCALAPPDATA% 部署副本
  notify/           WS 事件源（ws.rs 泛化 {path, handler, on_connect}，连 events.mux +
                    events.host 双下行）+ 帧分类（approval/question、turn/end 完成、
                    session/title 入 SessionBook 台账；子代理经 host 流 origin 过滤）；
                    sink 在 lib.rs：前台=任一窗口聚焦，按 settings.notify 三类规则门控
  theme.rs          标题栏主题跟随：轮询 dsh-home/settings.yaml 的 ui-theme.preference；
                    首启播种（settings.yaml 缺失时按系统深浅色预写 preference，dsh 缺省是浅色）
  progress.rs       首启进度模型：阶段权重、百分比映射、结构化 dsh-progress 负载
  tray.rs           系统托盘菜单（打开/诊断/技能管理/MCP 管理/重启/其它设置/退出）；
                    按需窗口 builder .center()（首启居中，记忆几何由 window-state 覆盖）；
                    diagnostics.rs 状态/日志环形缓冲；commands.rs 8 个 invoke 命令
  zoom.rs           UI 缩放：钩子脚本 hook_js(settings) 动态内嵌快捷键(on_page_load eval，
                    只注入 main 窗口)、direction 命令按设置读步进、ui-zoom.txt 持久化
  settings.rs       壳设置：settings.json 模型（步进 1-25%/快捷键/关窗行为/
                    notify 三类通知规则 {enabled,timing:background|always}（旧
                    notify_on_completion 读取时迁移进 turn_done）/提示音
                    silent|default|im|mail|reminder|sms|chime|drop|mellow/
                    check_update_on_launch 启动时检查更新（默认关））、校验、
                    落盘失败显式报错（不静默吞，防"保存成功但重启回退"）、
                    SettingsState、get/set_shell_settings + preview_completion_sound 命令
  skills.rs         技能管理：DSH_HOME(壳注入，非~/.dsh)的 skills/(启用) ↔
                    skills-disabled/(停用) 目录移动即开关（dsh watcher 热刷新）；
                    启动自动种子 ~/.dsh/skills(.skills-seeded marker 防复活)；
                    三源导入(codex/claude/opencode)+ZIP 本地导入
                    (inspect_zip_skills 识别包根 SKILL.md/顶层技能文件夹两种布局,
                    解包剥前缀+enclosed_name 防穿越+1万条目/256MB 上限;
                    前端经 tauri-plugin-dialog 选文件)、冲突覆盖/跳过、删除；
                    7 个命令 + SkillsHome 状态
  mcp.rs            MCP 管理：读写 dsh-home/profiles/web/cordis.patch.yml 中
                    name=='@deepseek-ai/dsh-mcp-client' 的 insert 条目（其余条目
                    Value 级保留，tmp+rename 原子写，BOM 容忍）；启停=entry 上
                    disabled:true（cordis loader 原生，HMR 热生效无需重启）；
                    编辑保留 toolCallTimeoutMs/reconnect 等高级键；启动种子
                    ~/.dsh 两层 patch（.mcp-seeded marker 防复活，源里 disabled
                    的不同步）；导入 claude(.claude.json)/codex(config.toml)/
                    opencode(opencode.json)，sse 不支持标记跳过；
                    6 个命令 + McpHome 状态
  update.rs         检查更新：GitHub releases/latest API（必带 UA；走系统代理——
                    外网场景，与回环必须 no_proxy 相反）、版本比较（自实现，
                    解析失败按非新版）、下载 *_x64-setup.exe 到系统下载目录
                    （.part→rename，百分比节流 emit update-download-progress）、
                    install_update 起 NSIS 后走 quit_app 自行退出——安装器是
                    本进程子进程，旧版钩子的 taskkill /T 会连它一起杀（立即安装
                    曾因此装不上）；本进程先死，钩子杀树即成空操作）、
                    open_update_page 用 rundll32 开 releases 页（不引 opener 插件）；
                    启动时检查默认关，有新版弹 toast，失败只记 events.log；4 命令
  remote/           远程访问：mod.rs=RemoteManager(生命周期/token/6 命令；reset_link
                    原地轮换 token 吊销泄露链接，域名不变) +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LBurny/deepseek-harness-desktop](https://github.com/LBurny/deepseek-harness-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
