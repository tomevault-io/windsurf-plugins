---
trigger: always_on
description: 本文件为 AI 编码代理提供本仓库的工作指引。
---

# AGENTS.md

本文件为 AI 编码代理提供本仓库的工作指引。

## 项目简介

Kimi Code Desktop:基于 [Kimi Code CLI](https://github.com/moonshotai/kimi-code) 的桌面客户端壳(Tauri v2 + React + TypeScript)。对话界面通过 iframe 内嵌官方 `kimi web` Web UI,壳自身提供用量统计、额度条、桌面通知、设置、托盘等能力。后端 `kimi web` 可运行在本机 / WSL / SSH 远端。

## 技术栈

- 前端:React 19 + TypeScript 6 + Vite 8 + Tailwind CSS 4 + zustand + lucide-react + @xterm/xterm(内嵌终端渲染)
- 后端:Rust(edition 2021, rust-version 1.77)+ Tauri v2 + tokio + reqwest(rustls)+ tokio-tungstenite + russh + keyring + portable-pty(内嵌终端 PTY)
- 主题:亮暗双主题,跟随官方 web UI(data-theme 属性切换,见下「约定」);色值全面对齐官方实测(浅色白底 + 官方蓝 #1783ff,深色纯中性灰 #121212 系 + #1a88ff)
- 字体:官方同款可变字体内嵌(src/assets/fonts,Schibsted Grotesk Variable 拉丁 + Noto Sans SC Variable 中文,均为 OFL 开源;栈与渲染参数见 theme.css;行标题 13px/475 对齐官方 .rlabel)

## 目录结构

```
src/                    渲染进程(React)
  components/           壳组件:ShellHome(视图容器,导航在标题栏;WebFrame 启动自动拉起——通道列表就绪后对激活通道自动 startChannel,每次启动一次、切通道不自动拉,本机缺 CLI 走既有 installConfirm 确认框(含「重新检测」按钮,覆盖自行 npm/brew 安装),开关在 设置→常规→本地服务,存 localStorage kimi.autoStartService 默认开,设置页停止服务仅当次会话有效;拉起决定时同步切 starting 防占位页按钮闪帧,取消安装落回 off;启动中界面为 BootTerminal 启动终端(server:launch 事件在 bootstrap 开始即下发将执行的命令行+注入 env[target.rs web_launch_display 与 web_command 同源,首选端口,无 token],打字动画与实际启动并行;动画收尾且 server:ready 后才切 iframe,RC 收养/设置页重启等无 starting 态场景不受门控直接切))/ TitleBar(铺平用量条(QuotaStrip)+ 对话/终端/统计/检查更新/主题切换/设置图标导航(官方同款黑底 tooltip,设置固定最右)+ 多通道切换器 + 窗口控制;主题切换经 pushThemeToFrames 反推官方 iframe,官方 MutationObserver 监听 data-color-scheme 无刷新跟随;检查更新按钮常驻,点击=手动检查,有新版出红点,点开 UpdateDialog(发现新版本/忽略此版本/下载安装))/ QuotaStrip(标题栏铺平式用量直显:实时指标胶囊 + 各窗口迷你额度计 + 钱包;纯展示,服务启停不在此——启动默认自动拉起/对话页占位图手动启动、启停走 设置→常规,评审结论:标题栏常驻开关的价值与显眼度不匹配)/ SkinStandee(实验性皮肤立绘,设置/统计/主页透出) / settings/ / pet/(桌宠窗口 PetWindow + 悬浮菜单 PetMenu)
  components/ui/        官方 kimi web 自研 ui-* 组件库(Vue)的 React 复刻:Select(fixed+portal 毛玻璃弹层、行首蓝对勾)、Switch(36×20)、Segmented(分段选择器,2-4 个短选项用;透明槽(不加背景色)+ 细边,elevated 选中面;白底页面上使用需经 className 补 bg-surface-tertiary 槽色)、Input/Textarea(inputCls/textareaCls 类串 + 组件;md 38px/sm 32px,0.5px border-strong 边,input-bg 底[浅纯白/深 10% 白]+ shadow-xs,focus = accent 边 + 3px accent-soft 环,hover 无变化);样式值实测自 CLI dist-web,新增表单控件一律用这套,不用原生 <select>/自绘开关/手写 input 类串。另有非复刻的自研组件:TomlHighlight(config.toml 源文件查看态的 TOML 语法高亮 + 行号,零依赖行级 tokenizer,编辑态仍为纯 textarea)。设置页卡片为官方填充式灰面板(components/settings/common.tsx 的 Card = surface-tertiary 无底边),面板内徽章/控件槽用 bg-fill、按钮/选中项用 bg-elevated;图表/明细表等数据可视化用 SurfaceCard(白底细边,灰面板会让图表发闷、热力图无色档融底)
  pages/                Onboarding / Settings(设置页「CLI 配置」组含远程协作分区 RemoteControlSettings:Remote Control 开关 + 访问链接/二维码面板,0.42 起 CLI 常驻解锁、从原 CLI 实验性分区拆出,开关存 desktop-config.json 的 remote_control 字段;「资源」组含插件分区 PluginsSettings:经 kimi web REST /api/v1/plugins* 管理插件,与 TUI /plugins 等效,安装/启停/移除均由 CLI 自身落盘;老版本 CLI 无此路由时提示升级)/ stats / terminal(内嵌终端 TUI 工作区:TerminalPage 网格容器 + TerminalPane xterm 封装 + workspace.ts 布局模型;无工具栏——新建经空槽内嵌项目选择面板(ProjectPicker,CLI workspaces.json 注册表,点击在该槽启动,默认目录降级幽灵行),布局经标题栏右键菜单(拆分/最大化/移入后台/布局模板/关闭;终端内容区右键还给 xterm/TUI,壳不占用)+ Ctrl+Shift+D/S 快捷键 + 拖拽(标题栏为 DnD source,落点中央=交换、30% 边缘带=移动到该侧 splitAt 插入——只重排不新建进程,拖到自身=回原位,预览为半槽高亮;拆分新建只经标题栏按钮/右键菜单/快捷键;拖拽幻影为小标签防「拖出应用」感);固定网格槽位 1–6 可见、总窗格 ≤12 超出进后台栏;关闭窗格自动 shrinkEmptyTracks 收缩全空行/列(拆分遗留空位自动去掉,至少 1×1;手动切模板的全空布局不动);右侧指令参考抽屉(CommandDrawer.tsx,推开式 300px,新手向官方斜杠命令/快捷键速查——按 CLI 0.42 官方文档静态精选,命令条目点击经窗格 onSessionChange 上报的会话映射 terminalWrite 插入当前活动终端[onActivate 记最近交互窗格],首次默认展开、记忆 kimi.termHelpOpen,入口=右缘把手/标题栏右键菜单);所有窗格恒定渲染在同一绝对定位层 key=paneId,交换/最大化只改 style 不卸载 xterm;输出经 tauri Channel<Vec<u8>> 二进制直发,布局持久化 kimi.termWorkspace、结构持久进程不持久,重启后可见窗格自动 respawn、后台惰性启动;xterm 配色固定深色)
  platform/kimi-api.ts  壳与渲染层的 API 契约(window.kimiApi)
  platform/tauri.ts     契约的 Tauri 实现(invoke / 事件监听)
  platform/os.ts        运行平台判定(IS_MAC/IS_WINDOWS,UA 方式;WSL 入口仅 Windows、协议 URL 分叉用)
  platform/protocol.ts  pet:// skin:// 自定义协议供图 URL 的平台分叉(Windows/Linux 为 http://<scheme>.localhost,mac 为 <scheme>://localhost;Rust handler 按 path 解析不受形态影响,CSP 两种形态均已放行)
  stores/ui.ts          界面状态(zustand)
src-tauri/src/          Rust 后端
  lib.rs                Tauri 入口与命令注册
  server.rs             spawn/管理 `kimi web` 进程,解析地址与 token;首选端口按构建类型分叉(release 58666 / dev 58766);
                        RC 开启时启动前做单例预检 rc_precheck(本机/WSL/SSH 通用;rc.json 持有者:健康则收养为后端[ServiceHandle::Adopted,
                        退出监控改 healthz 探活]、半死僵尸核验进程身份[basename 白名单 kimi/node,node 需命令行佐证]后按 pid 强杀、活但不可用出结构化冲突错误 RC_CONFLICT|,
                        前端"结束旧实例并重试"走 rc_kill_holder 命令);退出监控后移到服务就绪后启动(避免与早退探针竞报丢 stderr 尾部)
  rest.rs / ws.rs       REST 客户端与 WS 通知订阅器(/api/v1/*)
  cli.rs                CLI 自检测 / 安装 / 升级
  ssh.rs                进程内 SSH 客户端与端口转发
  config.rs / local_store.rs / target.rs   配置、本地数据直读、运行目标(本机/WSL/SSH);target.rs 另有功能开关注入:EXPERIMENTAL_FLAG_TABLE(对齐 CLI 0.42.0 FlagResolver 注册表,0.42 已删 SECONDARY_MODEL/REMOTE_CONTROL 实验 env)+ RUNTIME_SWITCH_TABLE(0.42 起常驻化的 search worker / minidb 读模型,对应官方 [database] 配置节),只注入与 CLI 默认不一致的项;启动时清理由 CLI 移除的旧 key 并把旧 RC env 迁移到 desktop-config.json 的 remote_control 独立字段(0.42 起 RC 常驻解锁,开启即启动 kimi web 附加 --remote-control,命令 remote_control_get/set)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yann-Up/kimi-code-desktop](https://github.com/Yann-Up/kimi-code-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
