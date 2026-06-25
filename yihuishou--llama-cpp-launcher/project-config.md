---
trigger: always_on
description: llama.cpp 桌面启动器。Rust + eframe/egui 0.34 GUI，管理 llama-server/RPC、预设系统、Windows 开机自启和快捷方式。单 binary，无
---

# LLama Launcher — Root

## 项目概述

llama.cpp 桌面启动器。Rust + eframe/egui 0.34 GUI，管理 llama-server/RPC、预设系统、Windows 开机自启和快捷方式。单 binary，无
Cargo workspace。

## 技术栈

- Rust 2021, egui 0.34, eframe 0.34
- serde + serde_json (配置/预设)
- rfd (文件对话框), log + env_logger, shortcuts-rs
- Windows优先: winit, winres, CREATE_NO_WINDOW

## STRUCTURE

```
root/
├── Cargo.toml              # 单 binary, llama_cpp_launcher.exe
├── build.rs                # Windows icon / manifest (winres)
├── src/main.rs             # 入口: eframe + CJK字体 + env_logger
├── src/app.rs              # LlamaLauncherApp: UI路由/菜单/状态/开机自启
├── src/config/             # AppSettings/Preset/GpuLayersMode/默认值与读写
├── src/engine/             # llama-server / rpc-server 进程管理与日志聚合
├── src/ui/                 # 7 个 egui 面板 (server/rpc/model/params/log/presets/cmds)
└── src/i18n.rs             # i18n::t(Key, lang)，zh/en key→文案映射
```

## WHERE TO LOOK

| Task                             | Location               | Notes                              |
|----------------------------------|------------------------|------------------------------------|
| UI面板/标签行为                        | src/ui/AGENTS.md       | 路由由 app.rs 控制，面板仅渲染                |
| Server/RPC生命周期、日志                | src/engine/AGENTS.md   | 状态机 + 进程管理 + log聚合                 |
| AppSettings/Preset/GpuLayersMode | src/config/settings.rs | 配置结构体 + Defaults + 读写              |
| i18n键映射 (zh/en)                  | src/i18n.rs            | Key enum + t() 函数，所有 UI 文本入口       |
| Windows快捷方式 (.lnk)               | src/shortcut.rs        | shortcuts-rs 封装                    |
| App结构、菜单与标签路由                    | src/app.rs             | LlamaLauncherApp::ui, tab_selected |

## CODE MAP (核心符号)

| Symbol                                                                | Type        | Location           | Role                                            |
|-----------------------------------------------------------------------|-------------|--------------------|-------------------------------------------------|
| main                                                                  | fn          | main.rs            | eframe 入口，加载字体 & env_logger                     |
| load_cjk_fonts                                                        | fn          | main.rs            | CJK字体加载（fallback）                               |
| LlamaLauncherApp                                                      | struct+impl | app.rs             | App根：settings/server/rpc/tab/lang/auto_start    |
| impl eframe::App for LlamaLauncherApp::ui                             | method      | app.rs             | 每帧 UI 路由 + 菜单栏                                  |
| enable_auto_start / disable_auto_start                                | fn          | app.rs             | Windows 注册表开机自启                                 |
| open_web_client_url, open_repo_url                                    | fn          | app.rs             | ShellExecuteW 打开浏览器                             |
| AppSettings                                                           | struct      | config/settings.rs | 全部配置字段（server/RPC/预设/GPU）                       |
| Preset                                                                | struct      | config/settings.rs | 可保存/应用/删除的预设；含 apply_to()                       |
| GpuLayersMode                                                         | enum        | config/settings.rs | Auto/All/Manual，序列化策略                           |
| SettingsManager                                                       | impl        | config/settings.rs | load/save + auto_detect_server_path/rpc         |
| ServerManager                                                         | struct+impl | engine/server.rs   | llama-server 生命周期、日志、launch_command             |
| RpcManager                                                            | struct+impl | engine/rpc.rs      | rpc-server 生命周期、连接状态                            |
| parse_tags                                                            | fn          | ui/model_panel.rs  | 文件名→9色彩色标签（参数量/量化/版本/训练方法/精度/LoRA/上下文长度/架构/模型名） |
| is_param_size, is_quantization, is_training_method, is_context_length | fn          | ui/model_panel.rs  | 标签分类判定辅助函数                                      |
| render_file_list                                                      | fn          | ui/model_panel.rs  | 按 FileMode(Main/Mmproj/Dflash) 过滤并渲染文件列表        |
| auto_detect_model_dir                                                 | fn          | ui/model_panel.rs  | 自动检测 model/models 目录（不区分大小写）                    |

## 模型标签系统（9 色方案）

`parse_tags()` 将文件名按 `-` 分段后匹配规则着色，渲染为圆角按钮：

| 颜色        | RGB           | 分类               | 判定函数/关键词                                                    | 示例                 |
|-----------|---------------|------------------|-------------------------------------------------------------|--------------------|
| 🟣 紫色     | (180,120,255) | 参数量              | `is_param_size` — 含数字+以 b/m 结尾                              | `7b`, `335m`       |
| 🟠 橙色     | (255,165,0)   | 量化类型             | `is_quantization` — q 后紧跟数字 / iq 后紧跟数字（排除 Qwen/QwQ）         | `q4_k_m`, `iq4_nl` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yihuishou/llama.cpp-launcher](https://github.com/yihuishou/llama.cpp-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
