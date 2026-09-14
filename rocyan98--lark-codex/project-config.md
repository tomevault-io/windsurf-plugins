---
trigger: always_on
description: [面向用户的 README](README.md)
---

# Lark-Codex：Agent 安装与使用指南

[面向用户的 README](README.md)

本指南仅在**用户请求安装、配置、排障或使用 Lark-Codex** 时适用。读取本文件本身不触发安装、配置修改或任务执行，也不要求处理源码的 Agent 先安装应用；既有上级及宿主规则继续适用。

以下操作针对已发布的 macOS 应用，无需克隆源码或安装开发环境。命令与界面以实际安装版本为准；命令不匹配时先按第 5 节定位包装器并查看帮助，不要猜测接口。

## 1. 执行边界

- 根据用户当前请求推进安装或操作；可逆准备和只读检查不必重复确认。只有缺少具体决定或授权时才询问。
- 保留现有数据、配置和未完成任务。不要退出 Codex Desktop、接管已有 Codex 会话，或为验证安装而启动真实任务。
- 优先通过 Lark-Codex 的「连接配置」「端口设置」「使用引导」操作；当前没有对外承诺的无头配置 CLI。
- App Secret、完整 frpc.toml（可能含隧道 token）、CLI 会话文件、内部令牌和 `runtime.json` 不得输出到对话、日志或 Issue。请用户在应用界面输入凭据；不要索要密码、复制飞书客户端 token 或关闭脱敏。
- 创建任务、评论、执行任务、响应审批、删除数据等写操作必须符合用户请求，并使用真实飞书用户授权。安装授权本身不包含这些业务操作。

## 2. 确认设备、版本和已有安装

当前发布包适用于 **Apple Silicon（arm64）、macOS 13 或更新版本**，不适用于 Intel Mac 或 Windows。读取：

```sh
uname -m
sw_vers -productVersion
```

如果 `uname -m` 返回 `x86_64`，继续用 `sysctl -n hw.optional.arm64` 判断是否为 Rosetta 下的 Apple Silicon；结果为 `1` 才支持当前 arm64 包。确认前不要安装。

检查 `/Applications/Lark-Codex.app` 和以下目录是否存在，只读取必要元信息，不展示私密文件内容：

```text
~/Library/Application Support/Lark-Codex/
├── data/       数据库、附件、运行时信息
├── secrets/    飞书凭据、frpc 配置、内部令牌
├── deploy/     本机端口配置
└── caddy/      证书与 Caddy 状态
```

存在数据目录就按升级处理；不要清空目录或覆盖成发布者的配置。首次升级启动时，若新目录不存在且旧版已退出，应用会自动迁移旧版数据目录；新旧目录同时存在或目录受其他进程占用时按提示处理，不手工合并或覆盖。正常替换 `.app` 保留这些数据。需要备份时先确认备份位置及访问权限；复制整个数据目录应先正常退出 Lark-Codex，避免只复制正在写入的 SQLite 主文件。

应用内置 Node、后端、网页前端、Codex 桥接、SQLite 组件、Caddy 和 frpc 客户端。用户仍需安装并登录 Codex 和飞书客户端，并准备自己的飞书应用和公网 frp 服务。**不需要另外安装 Node、Docker、Rust 或 Homebrew。**

## 3. 下载、校验与安装

1. 从用户提供的 GitHub 仓库或 Release 页面读取版本、资产名和实际下载 URL。仓库未明确时只询问仓库链接；不要猜测 owner/repo 或拼接未经核对的资产地址。
2. 下载同一 Release 的 `macos-arm64.dmg` 及对应 `.sha256` 文件。避免把自动生成的 Source code 压缩包当作安装包；预发布版本需符合用户选择。
3. 将两个文件保存在同一目录，核对 SHA-256。例如，实际文件名与下面完全一致时执行：

   ```sh
   cd "$HOME/Downloads"
   shasum -a 256 -c "Lark-Codex-0.1.1-macos-arm64.dmg.sha256"
   ```

   其他版本使用实际文件名。检查校验文件中的文件名与下载的 DMG 一致；必须得到 `OK`。失败就停止安装并重新核对下载来源，不修改校验值来通过检查。校验和用于检查文件完整性，不等于 Apple 公证或发布者身份认证。

4. 升级前通过菜单栏「退出 Lark-Codex」或 Command-Q 正常退出旧版，等待它启动的服务停止。关闭窗口只会隐藏应用，不能作为退出判断；不要使用范围过大的 `pkill node` 或结束 Codex。
5. 挂载 DMG，将完整的 `Lark-Codex.app` 拖入「Applications」。已有旧版时替换整个应用包，不合并内部文件，也不从挂载卷直接作为长期安装运行。
6. 从 `/Applications/Lark-Codex.app` 启动应用，然后推出 DMG。

当前版本使用本地 ad-hoc 签名，尚无 Developer ID 签名及 Apple 公证。若 macOS 拦截，说明当前状态与实际提示，请用户在确认来源后通过系统「隐私与安全性」处理首次打开。此系统安全决定交给用户；不要自动清除 quarantine、关闭 Gatekeeper 或执行绕过系统保护的命令。若系统报告恶意软件或文件损坏，应停止并核对原包，不能把所有拦截都归因于未公证。

### 应用内更新

已有版本可通过「应用设置 → 应用更新」或菜单栏「检查更新…」检查版本，读取更新说明后按用户授权下载。下载过程会验签；只有确认「安装并重启」才会停止本机服务并替换应用。先处理正在执行的任务和未保存配置，不能把“检查更新”的授权当作“现在重启”的授权。

不要自行下载 `.app.tar.gz` 后绕过更新器解压覆盖，也不要修改 `latest.json` 或签名让检查通过。自动安装失败时，根据实际错误改用同版本 DMG，保留数据目录；不要结束 Codex Desktop。

## 4. 用应用界面完成配置

### 公网隧道和端口

打开「端口设置」或「使用引导」，读取**当前 Caddy 端口**，不要把文档默认端口当作当前值。在「连接配置」粘贴服务商或用户自行准备的完整 frpc.toml，保留其服务器地址、控制端口与认证参数。

使用引导固定为三步：配置 frp 客户端、创建飞书应用、确认 Codex 登录。DNS 解析和公网访问检查位于第一步内；HTTPS/HTTP 域名模式显示解析信息，TCP 公网 IPv4 模式显示无需 DNS。TCP 仍需验证公网访问。

公网地址只从 frpc.toml 推导。必须且只能有一个启用的 HTTP、HTTPS 或 TCP 代理匹配 `127.0.0.1`（或 `localhost`）及当前 Caddy 端口；修正匹配关系，不增加手工公网地址文件。

| 隧道类型 | 公网入口规则                 | 配置要点                                                                      |
| -------- | ---------------------------- | ----------------------------------------------------------------------------- |
| `https`  | `https://域名`，公网 443     | `customDomains` 中只有一个完整域名；443 能到达本机 Caddy，由 Caddy 管理证书   |
| `http`   | `http://域名`，公网 80       | 协议取自 `type`，域名本身不能决定 HTTPS                                       |
| `tcp`    | `http://公网IPv4:remotePort` | `serverAddr` 必须是公网 IPv4；不填写 `customDomains` 或 `subdomain`，无需 DNS |

HTTP/TCP 模式明文传输登录会话与业务数据；准备正式使用时优先配置 HTTPS。当前应用按 HTTP/HTTPS 标准公网端口工作，不从 frpc 控制端口推测网页端口。公网 frp 服务端不随应用安装。

域名模式按隧道服务商实际提供的入口设置 A/CNAME；不能仅因 `serverAddr` 是控制服务器就认定它一定是 DNS 目标。代理应直接转发到本机 Caddy，不依赖 `includes`、`store.path`、`subdomain` 或代理插件来推导入口。

修改本机端口用「端口设置」保存：四个本机端口必须互不相同，Caddy 端口变化会同步对应隧道的 `localPort`。不要误改 frpc 的 `serverPort`。

### 飞书应用

1. 由用户在飞书开放平台创建企业自建网页应用，在「连接配置」输入该应用的 App ID、App Secret。
2. 先检查当前 frpc 表单，让「使用引导」生成该用户自己的地址；不要沿用截图、历史部署记录或发布者的域名。
3. 按引导的复制按钮填写桌面端主页、移动端主页、同源 H5 可信域名和重定向 URL。重定向 URL 保留引导提供的末尾 `/`，不要自行猜测 `/callback` 路径。
4. 申请「获取用户 user ID」权限 `contact:user.employee_id:readonly`，完成开放平台版本发布，并设置需要使用该应用的人员可用范围。
5. 用户在飞书客户端登录；部署就绪后，通过 Lark-Codex「使用引导 → 打开飞书验证」进入自己的飞书应用并验证实际登录。

App ID/App Secret 检查仅验证凭据；不能据此判断应用已发布、可用范围正确或用户登录已成功。当前没有应用发布状态查询能力，这些项目需要通过飞书后台和实际登录确认。

### Codex 与保存生效

请用户安装并登录 Codex。Lark-Codex 自动检测本机 Codex 程序和登录状态，项目来自 Codex Desktop 项目列表。需要添加项目时在 Codex Desktop 中完成；不要尝试用 taskctl 创建或注册项目。

「使用引导」检查的是当前表单，不自动保存、不自动重启。按结果修正后保存配置，再按用户当前意图选择立即重启或稍后重启；稍后重启时运行服务仍使用旧配置。相同配置无需反复保存和重启。

只有服务启动、当前配置生效后，再做公网访问和真实飞书登录验证。Codex 登录状态检查不证明在线令牌有效、额度充足或真实任务执行成功；没有执行授权时将真实执行标为「未验证」。

## 5. 安装 Skill 并调用内置 taskctl

首次启动且尚未安装配套技能时，「让 Codex 使用 Lark-Codex」提示提供「安装到 Codex」和稍后选项；之后可通过「应用设置 → Agent Skill」安装、更新或重新检查。首版仅支持 Codex，默认将 `manage-lark-codex` 安装到 `~/.agents/skills/`。

安装只写入技能文件，不包含 CLI 身份授权。已有用户修改时，只有用户明确选择「使用随包版本」才替换；符号链接或受其他工具管理的目录不直接覆盖。若已有旧版名称 `manage-lark-taskboard` 的技能，或旧 `~/.codex/skills`、`$CODEX_HOME/skills` 中已有同名技能，按提示在原位置或管理器处理，不再复制第二份，也不自动改名。应用升级仅提示技能可更新，不静默覆盖。

成功状态只证明文件已安装。先在 Codex 技能列表确认，未识别时由用户按需强制重新加载技能或重新打开 Codex，再在新任务核验；不要为验证而退出 Codex Desktop、接管已有会话或启动真实业务任务。

CLI 示例使用该技能中的包装器，直接调用完整 `.app` 内的 Node 和 taskctl，不依赖源码目录或全局 Node。默认安装时：

```sh
TASKCTL="$HOME/.agents/skills/manage-lark-codex/scripts/taskctl.sh"
"$TASKCTL" --help
"$TASKCTL" health

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RocYan98/Lark-Codex](https://github.com/RocYan98/Lark-Codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
