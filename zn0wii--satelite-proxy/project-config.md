---
trigger: always_on
description: 面向 AI agent 的项目速查文档。读完本文即可定位绝大多数代码，无需重复探索。
---

# AGENTS.md — Satelite Proxy 项目地图

面向 AI agent 的项目速查文档。读完本文即可定位绝大多数代码，无需重复探索。
最后核对：2026-09-13（**Xray 传输面修正 + 真内核矩阵测试**：① h2/http 传输已被 Xray v26 配置层移除（`network:"http"` 拒载）——Xray 生成期报错跳过 + `supports_node` 列表过滤，sing-box/mihomo 原生可用；② xhttp `extra` 调优参数接入：分享链 base64url `extra=`（裸 JSON 亦收）与 clash `xhttp-opts.extra` 解析入 `Transport::Xhttp.extra`，原样嵌入 `xhttpSettings.extra`，非法 JSON 丢弃防整配置拒载（v2rayN 对齐，edgetunnel xhttp 场景）；③ 新增真内核 live 验证 `live_share_link_pipeline_validates`（真实分享链接全管线）与 `live_all_protocol_matrix_validates`（全协议×传输×TLS 矩阵 + Valid/GenReject/CoreReject 预期表），见 §8。**智能切换二次重构**：`smart_switch.rs` 新增出口巡检（每 tick 经内核 URL 探测当前出口、快慢双档复核防抖，死出口不依赖用户流量即可发现）+ 恢复扫描（死出口绕过 dwell/cooldown：ping 分批排序 → top3 经内核 delay API 实测验证 → 取最低验证延迟者热切；ping/verify 失败递增弹出 30s→30min），URL 延迟为唯一对比口径、绝不凭 ping 上位，见 §5.1。此前 2026-09-12：**MASQUE 接入 + 三核多副进程**：新增 `Protocol::Masque`（mihomo 独占，usque 式 ECDSA 密钥对；mihomo 主内核原生可用），副进程机制从单一 Xray 泛化为 **Xray + mihomo 双副进程**（`SidecarPlan` 条目带 kind、单一连续端口空间、看门狗按副进程独立跟踪），masque 节点在 sing-box 主核下钉选到 mihomo 副进程经 loopback socks 委托，见 §5.4/§5.6/§9.20。窗口两模式开放自由放大：pro ≥960×720 / simple ≥320×480 无上限，经 `useViewportScale` 等比缩放，尺寸按模式持久化，见 §4/§9.6。此前 v1.0.9，三内核：sing-box / Xray / mihomo；新增首页「网络探测」卡=延迟+出口 IP 竞速探测，见 §5.6/§5.8/§6.3。内核意外退出修复：watchdog 真重启 + `core-status-changed` 事件 + 启动就绪须实测 mixed 端口拨号，见 §5.1/§5.6/§6.2。智能切换被动检测 2026-09 重构：失败判定=≤3s 快死或 ≤15s 零字节（拨号超时带），mihomo 增设内核日志流 `/logs` WS 监听（§5.1 `log_listener.rs`）；同批性能改造=快照流三档变速（需求心跳+TUN 退避）与 passive stats 单趟化，见 §5.1。CI 与本地打包脚本默认改为三内核四平台全打包，Linux 补齐 xray/mihomo 基础设施，mihomo geodata 改为固定快照机制，sing-box 版本号三处统一为 v1.13.18，见 §9.19；mihomo 的 amd64 发行包改用 `-compatible`（GOAMD64=v1）变体，修复 Intel 版应用在 Rosetta 2 / 老 Intel CPU 上安装 mihomo 即 fatal 的问题，见 §9.17⑦。远程加密 DNS 开放用户自定义：`DnsSettings.remote_dns` 非空即整体替换内置池，三生成器+诊断统一走 `effective_remote_pool()`，见 §18①。规则集「指定」支持多选节点池：`RuleSet.node_ids`（2+ 勾选=整组显式池，与 Filter 池同 tag `smart-<集id>`、同生成/维护路径，勾 1 个仍走存量单钉 `node_id`），见 §5.3/§5.4/§5.5）。

## 0. 阅读与维护规则（必读）

**对 agent 的要求**：在本仓库做任何改动前，先通读本文档——尤其是「§1 快速上手」、§7 修改场景速查表、§9 约定与坑。不要凭猜测探索全库。

**文档同步规则**：项目发生重大变动时，**必须与代码同一次提交同步更新本文档**，包括但不限于：

| 变动类型 | 需更新的章节 |
|---|---|
| 新增 / 删除 / 移动模块或源文件 | §3 目录速览、§5/§6 对应模块详解 |
| 新增 / 改名 command 或 Tauri 事件 | §5.1（注册表）、§5.8、§6.2 |
| 数据模型 / 存储结构 / 磁盘布局变化 | §5.2、§5.3 |
| 配置生成或内核管理方式变化 | §5.4、§5.5 |
| 构建 / 打包 / 测试流程变化 | §1 快速上手、§8 |
| 新的平台分支、新的坑 | §9 |

小改动（文案、bugfix、样式微调）不强制更新；文中行数标注允许过时，以「文件存在性与职责描述」为准，发现明显过时顺手修正并更新文首「最后核对」日期。

### 更新说明输出模板（用户要求的固定格式）

用户让「生成更新说明 / 改动摘要」时，**必须按以下模板输出**（更新说明 4–5 条精炼要点 + 固定的下载指南与 macOS 隔离提示）：

````markdown
### 更新说明

- **<要点>**：<一句话说明>
- …（4–5 条，重要功能加粗开头）

### 下载指南

* **macOS Apple Silicon（M 系列）** → `aarch64.dmg`
* **macOS Intel** → `x64.dmg`
* **Windows 64 位** → `x64-setup.exe`
* **Linux 64 位** → `amd64.AppImage`

根据你的系统和 CPU 架构选择对应安装包即可。


**macOS 如果提示「Satelite 已损坏，无法打开」**，可在终端执行：

```bash
sudo xattr -d com.apple.quarantine /Applications/Satelite.app
```
````

## 1. 快速上手：环境 · 编译 · 测试 · 打包

### 环境要求

- **Node + pnpm**（registry 已锁定 npmmirror，见 `.npmrc`；依赖只能用 pnpm 装）
- **Rust stable**：Windows 需 MSVC 工具链（build 脚本用 vswhere 检测）；macOS 需 Xcode CLT
- 平台限制：DMG 只能在 macOS 打，Windows 安装包只能在 Windows 打；Apple Silicon 可交叉编 Intel（脚本自动 `rustup target add x86_64-apple-darwin`）

### 开发调试

```bash
pnpm install        # 安装前端依赖
pnpm tauri dev      # 一键启动 Rust 后端 + WebView（Vite 端口 1420 strictPort）
```

- 前端改动走 HMR；Rust 改动自动重编并重启应用
- **不要用 `pnpm dev` 调 UI**——只起 Vite 没有后端，所有 `invoke` 会失败；UI 调试也用 `pnpm tauri dev`
- 首次 dev 缺 sing-box 内核 / 内置规则集会**自动联网下载**；离线环境先跑下面的「资源预取」
- 深链调试： schemes 为 `clash://` `sing-box://` `singbox://`（Windows/Linux dev 下启动时自动注册）

### 检查与测试

```bash
pnpm build                                        # 前端：tsc 严格类型检查 + vite 产物（提交前必过）
cd src-tauri && cargo check                       # Rust 快速检查
cd src-tauri && cargo test                        # Rust 全部测试（含散落 #[cfg(test)] 单测）
cd src-tauri && cargo test --test parse_subscription          # 只跑订阅解析集成测试
cd src-tauri && cargo test --test download_core_live -- --ignored  # 真网下载 live 测试（慢，慎跑）
cd src-tauri && cargo fmt / cargo clippy          # 标准 Rust 工具链
```

- 前端**没有** ESLint/Prettier 配置，质量门槛 = `tsc` strict + `pnpm build`
- 测试 fixtures 在 `src-tauri/tests/fixtures/`（clash yaml ×2、singbox json ×1）

### 打包发布

```bash
# macOS DMG（产物: src-tauri/target/<aarch64|x86_64>-apple-darwin/release/bundle/dmg/）
./scripts/build-dmg.sh                        # 按本机架构，三内核（默认）
./scripts/build-dmg.sh --arch arm64
./scripts/build-dmg.sh --arch intel           # 交叉编译；等价 build-dmg-intel.sh
./scripts/build-dmg.sh --singbox-only         # 瘦身：只打 sing-box（缺失自动 fetch）

# Windows（产物: src-tauri/target/release/bundle/nsis/ 或 .../msi/）
pwsh scripts/build-windows.ps1                        # NSIS 安装包，三内核（默认）
pwsh scripts/build-windows.ps1 -Bundle msi            # MSI
pwsh scripts/build-windows.ps1 -SingboxOnly           # 瘦身：只打 sing-box（缺失自动 fetch）

# Windows 便携版（产物: src-tauri/target/release/bundle/portable/Satelite_<版本>_x64_portable.zip）
pwsh scripts/build-windows.ps1 -Bundle portable       # 解压即用 zip：exe + resources/ + portable.flag（见 §9.19），三内核
pwsh scripts/build-windows.ps1 -Bundle portable -SingboxOnly  # 单内核便携版

CI（`.github/workflows/release.yml`）只打三内核版（sing-box + Xray + mihomo，含各自 geodata；`tauri.conf.json` 的 bundle targets 已收窄为 app/dmg/nsis/appimage——不再产 MSI/deb/rpm，本地脚本传 `--bundles` 仍可覆盖），

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zn0wii/satelite-proxy](https://github.com/zn0wii/satelite-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
