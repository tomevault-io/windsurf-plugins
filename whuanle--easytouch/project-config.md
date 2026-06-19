---
trigger: always_on
description: 跨平台系统自动化操作工具，支持 Windows、Linux、macOS。提供 CLI 命令行和 MCP 服务器两种使用方式，支持鼠标键盘控制、屏幕截图、窗口管理、系统信息查询等功能。
---


# EasyTouch Skill

## 环境要求

使用 npm 安装时：Node.js >= 18（npm 安装场景）

>  直接从 Github Release 下载文件后添加环境变量。



## 安装方式

### npm 安装

```bash
# 推荐：自动匹配当前平台
npm install -g @whuanle/easytouch

# Windows
npm install -g easytouch-windows

# Linux
npm install -g easytouch-linux

# macOS
npm install -g easytouch-macos
```



安装后命令入口：

```bash
et help
```



## CLI 命令

### 核心命令

```bash
# 显示命令总览与参数格式
et help

# 显示当前运行时状态与主机平台信息
et status

# 列出各平台实现状态与能力模块
et platforms

# 显示 CLI/MCP 接口与能力映射概览
et interfaces

# 显示能力接入和运行时要求清单
et requirements

# 启动 MCP stdio 服务
et mcp-stdio

# 输出 MCP manifest（工具清单）JSON
et mcp-stdio --output json
```



### 自动化命令

默认命令执行输出结果内容为 json，如需直接输出文本可使用 `-output text` 指定。



#### 系统信息

| 命令 | 用途 |
| --- | --- |
| `system os-info` | 读取操作系统版本、架构、主机名等信息 |
| `system cpu-info` | 读取 CPU 架构与核心信息 |
| `system memory-info` | 读取内存总量、可用量与占用情况 |
| `system disk-list` | 列出磁盘与容量信息 |
| `system process-list` | 列出当前进程列表 |
| `system hardware-info` | 读取硬件概览（架构、核心、页大小、物理内存、虚拟内存、机器名） |
| `system network-info` | 读取网络适配器信息（IPv4、MAC、网卡类型、DHCP 状态） |

平台说明：

- Windows：全部可用。
- Linux：整体可用，`system network-info` 依赖 `ip` 命令。
- macOS：整体可用，信息主要来自 `sysctl`、`vm_stat`、`ifconfig` 等系统命令。



#### 鼠标操作

| 命令 | 用途 |
| --- | --- |
| `mouse position` | 读取当前鼠标坐标 |
| `mouse move --x <x> --y <y> [--duration-ms <ms>] [--jitter-px <px>] [--step-delay-ms <ms>]` | 以渐进轨迹移动鼠标，支持抖动与延迟，更接近人类操作 |
| `mouse click [--button <left\|right\|middle>] [--count <n>]` | 执行鼠标点击 |
| `mouse scroll --delta <amount>` | 执行鼠标滚轮滚动 |

平台说明：

- Windows：全部可用。
- Linux：依赖 `xdotool`，并要求运行在 X11 或 XWayland 会话中；Wayland 原生会话下可能被桌面环境限制。
- macOS：依赖系统 Accessibility 权限；未授权时鼠标读取和注入会失败。



参数说明：

- `--x` / `--y`：目标坐标（整数）
- `--duration-ms`：总移动时长（毫秒），默认 `280`
- `--jitter-px`：轨迹抖动幅度（像素），默认 `3`
- `--step-delay-ms`：每一步移动间隔（毫秒），默认 `8`
- `--button`：按键类型，`left`、`right`、`middle`
- `--count`：点击次数，默认 `1`
- `--delta`：滚动量，正负值分别代表不同方向



#### 窗口与应用

| 命令 | 用途 |
| --- | --- |
| `window list [--include-hidden] [--pid <pid>]` | 列出窗口，可按可见性和进程过滤 |
| `window foreground` | 读取当前前台窗口信息 |
| `window find --title <text> [--match <contains\|exact>] [--include-hidden] [--pid <pid>]` | 按标题查找窗口 |
| `window activate --handle <handle>` | 按句柄激活窗口 |
| `window show --handle <handle>` | 显示窗口并请求激活 |
| `window minimize --handle <handle>` | 最小化窗口 |
| `window maximize --handle <handle>` | 最大化窗口 |
| `window restore --handle <handle>` | 恢复窗口（从最小化/最大化） |
| `window move --handle <handle> --x <x> --y <y> [--width <n>] [--height <n>]` | 拖曳/移动窗口到指定坐标，可选调整尺寸 |
| `window close --handle <handle>` | 按句柄请求关闭窗口 |
| `app launch --target <path-or-uri>` | 启动应用、打开文件或 URI |

平台说明：

- Windows：全部可用。
- Linux：`window list`、`window foreground`、`window activate`、`window close` 可用；`window show`、`window minimize` 依赖 `xdotool`，`window maximize`、`window restore`、`window move` 依赖 `wmctrl` 和兼容 EWMH 的窗口管理器。
- Linux：整体要求 X11 或 XWayland 会话，Wayland 原生会话下窗口句柄和控制能力可能受限。
- macOS：窗口控制依赖 Accessibility 和 Automation 权限；`window show`、`window minimize`、`window maximize`、`window restore`、`window move`、`window close` 通过 `System Events` 操作，若同一进程里有多个同名窗口，系统会按首个匹配窗口执行。

参数说明：
- `--title`：窗口标题关键字
- `--match`：匹配模式，`contains` 或 `exact`
- `--include-hidden`：包含隐藏窗口
- `--pid`：按进程 ID 过滤
- `--handle`：窗口句柄（非负整数）
- `--x` / `--y`：窗口目标位置
- `--width` / `--height`：窗口目标尺寸（可选，不传则保持原尺寸）
- `--target`：可执行路径、文件路径或 URI



#### 剪贴板

| 命令 | 用途 |
| --- | --- |
| `clipboard get-text` | 读取剪贴板文本 |
| `clipboard set-text --text <value>` | 写入剪贴板文本 |
| `clipboard get-files` | 读取剪贴板中的文件列表 |
| `clipboard set-files --paths <path1;path2;...>` | 写入剪贴板文件列表（用于后续粘贴文件） |
| `clipboard set-image --path <image-file>` | 写入剪贴板图片（用于后续粘贴图片） |

平台说明：

- Windows：全部可用。
- Linux：依赖 `wl-copy` / `wl-paste` 或 `xclip` / `xsel`；缺少这些工具时，对应文本、文件、图片剪贴板能力不可用。
- macOS：文本剪贴板通过 `pbcopy` / `pbpaste`；文件和图片剪贴板通过 `osascript` / JXA 写入，若宿主进程缺少 Automation 权限可能失败。

参数说明：
- `--text`：要写入的文本内容
- `--paths`：多个文件路径，使用分号 `;` 分隔
- `--path`：单个图片文件路径



#### 键盘操作

| 命令 | 用途 |
| --- | --- |
| `keyboard key --key <name>` | 发送单个按键 |
| `keyboard hotkey --keys <combo>` | 发送组合键 |
| `keyboard type --text <value>` | 直接注入文本内容（直输模式） |
| `keyboard type-keys --text <value> [--key-delay-ms <ms>]` | 按键位逐字输入（模拟人类打字模式） |
| `keyboard ime-switch [--strategy <win-space\|alt-shift\|ctrl-shift>]` | 切换输入法 |
| `keyboard caps-lock [--state <toggle\|on\|off>]` | 控制大小写状态 |
| `keyboard paste [--expect-title <text>] [--match <contains\|exact>]` | 发送粘贴动作，可加窗口标题保护 |

平台说明：

- Windows：全部可用。
- Linux：依赖 `xdotool`，并要求运行在 X11 或 XWayland 会话中；Wayland 原生会话下键盘注入通常不稳定或被系统阻止。
- macOS：依赖 Accessibility 权限；`keyboard ime-switch` 的策略会映射到系统快捷键，是否真正切换输入法取决于系统当前输入源绑定。
- macOS：`keyboard caps-lock` 目前仅稳定支持 `toggle`，`on` / `off` 仍不保证可用。

参数说明：
- `--key`：按键名（如 `enter`、`esc`、`f5`）
- `--keys`：组合键（如 `ctrl+c`、`ctrl+shift+s`）
- `--text`：输入文本
- `--key-delay-ms`：模拟键位输入时每个按键之间的延迟
- `--strategy`：输入法切换快捷键策略
- `--state`：大小写状态，`toggle`、`on`、`off`
- `--expect-title`：执行粘贴前要求前台窗口标题匹配
- `--match`：标题匹配模式，`contains` 或 `exact`



#### 屏幕与截图

| 命令 | 用途 |
| --- | --- |
| `screen displays` | 列出显示器与分辨率信息 |
| `screen pixel-color --x <x> --y <y>` | 读取指定像素颜色 |
| `screen capture [--path <file>] [--display-id <id>] [--window-handle <handle>]` | 默认截取所有屏幕，或按屏幕/窗口定向截图 |

平台说明：

- Windows：全部可用。
- Linux：`screen displays` 依赖 `xrandr`，`screen capture` 依赖 `grim`、`gnome-screenshot`、`scrot` 或 `import` 之一。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whuanle/EasyTouch](https://github.com/whuanle/EasyTouch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
