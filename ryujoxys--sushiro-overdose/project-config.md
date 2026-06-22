---
trigger: always_on
description: > 本文件专为 AI 编码助手 (Cursor / Claude / Copilot 等) 编写。
---

# AGENTS.md — LLM 专用项目文档

> 本文件专为 AI 编码助手 (Cursor / Claude / Copilot 等) 编写。  
> 人类开发者请看 `README.md`。

---

## 项目概述

**sushiro-overdose** 是寿司郎 (SUSHIRO) 餐厅的自动预约抢号工具。

核心流程：本地 MITM 代理拦截 PC 微信小程序流量 → 捕获凭证参数 → 直连官方 API 轮询/抢号。

**技术栈**：Go 1.23，零外部依赖（纯标准库），单 `package main`。

**目标平台**：macOS (amd64/arm64 Universal)、Windows (amd64/arm64)、Linux (amd64/arm64)。

---

## 架构概览

```
用户双击运行
    │
    ▼
main.go (默认启动 Web UI)
    │
    ├── web.go          HTTP 服务器 127.0.0.1:8081
    │   ├── web_handlers.go   REST API + SSE
    │   └── web_static.go     内嵌 HTML/CSS/JS 单页应用
    │
    ├── engine.go       后台引擎（可从 Web 控制）
    │   ├── 捕获模式: proxy.go + cert.go + platform_*.go
    │   └── 抢号模式: api.go + preferences.go
    │
    └── CLI 子命令 (cli/calendar/sniper/list/cancel/...)
```

### 两种使用模式

1. **Web UI 模式（默认）**：无参数运行 → 启动 HTTP 服务 → 优先打开独立应用窗口，失败时回退默认浏览器
2. **CLI 模式（高级）**：`sushiro cli` → 传统终端交互

---

## 文件清单与职责

### 核心入口

| 文件 | 职责 |
|------|------|
| `main.go` | 程序入口、CLI 命令分发、前台 CLI 流程、`runBookingLoop` 抢号循环 |
| `daemon.go` | 后台启动/停止/status、守护进程子进程与 PID 读写 |
| `engine.go` | **Web 控制的后台引擎**：管理捕获/抢号生命周期，状态广播到 SSE，可启动/停止 |
| `engine_sniper.go` | Web 狙击计划执行引擎 |

### Web UI

| 文件 | 职责 |
|------|------|
| `web.go` | HTTP 服务器启动，端口冲突自动换端口，Settings 注入 |
| `web_handlers.go` | Web 通用 handler/helper 与首页 |
| `web_calendar.go` | 日历/门店 API |
| `web_engine.go` | 状态、预约、引擎控制、洞察 API |
| `web_preferences.go` | 偏好、通知、repair/uninstall API |
| `auth_import.go` | 手动导入凭证 API：解析手机抓包导出的 JSON/curl/raw headers 并保存凭证参数 |
| `web_sniper.go` | Web 狙击计划 API |
| `web_sampling.go` | Web 信息收集 API |
| `mobile_auth_capture.go` | 手机凭证捕获 API：局域网引导页 + 手机代理捕获真实微信凭证参数 |
| `web_queue_trends.go` | 本地到店预测 API |
| `web_queue_live.go` | 实时排队 API（公开门店等位/区域/单店详情） |
| `web_cloud_auth.go` | 云端数据登录 API：Worker URL 配置、GitHub OAuth 本机回调、会话验证/退出 |
| `web_events.go` | SSE 事件总线 |
| `web_static.go` | `sushiroLogoSVG` Logo SVG 常量 + `indexHTML` 完整单页（Sushiro 品牌配色 + 官网同款布局） |

### API 与数据

| 文件 | 职责 |
|------|------|
| `api.go` | `Client` — 寿司郎官方 API 封装（门店/时段/创建预约/取消预约） |
| `queue_live.go` | 公开排队接口客户端：门店列表、单店排队、区域列表（标准库实现，支持 `SUSHIRO_TOKEN` 覆盖）；解析 `getStoreById` 的 `groupQueues` 得到当前叫号 |
| `queue_live_panel.go` | 单店实时面板聚合：实时叫号/在等桌数/预估等待 + 由本机采样历史算近15分钟叫号与历史均速 |
| `queue_alerts.go` | 叫号提醒规则与去重状态：`wait_below`（预估等待降到阈值）/`called_reach`（叫号接近手中号），采样循环命中即经通知渠道推送 |
| `cloud_auth.go` | 云端数据配置与客户端：本地只保存 Cloudflare Worker URL 和应用 session，不保存 Turso token |
| `config.go` | `Settings` 结构体定义，`LoadSettings` 从 JSON 文件加载（备用，当前未被调用） |
| `tokens.go` | 捕获到的凭证参数模型、本地配置读写、旧配置迁移、凭证参数 → `Settings` 转换 |
| `preferences.go` | **用户偏好持久化**：人数/桌型/自定义时段范围/日期与时段优先级，存到 `~/.sushiro/preferences.json` |
| `slot.go` | `Slot`/`StoreInfo`/`ReservationRecord` 数据结构，时间格式化工具 |

### 代理与捕获

| 文件 | 职责 |
|------|------|
| `proxy.go` | MITM 代理服务器、请求解析捕获、门店选择、旧版时段配置 |
| `cert.go` | CA/叶子证书生成，存储路径 `~/.sushiro-proxy/` |
| `watchdog.go` | `proxy_active.json` — 异常退出后清理残留系统代理 |

### 平台适配

| 文件 | 职责 |
|------|------|
| `platform.go` | 跨平台函数转发（大写导出 → 小写平台实现） |
| `platform_darwin.go` | macOS：`networksetup` 代理、`security` 证书、`osascript` 通知 |
| `platform_windows.go` | Windows：注册表代理 + `InternetSetOption` 刷新、`certutil` 证书、PowerShell 通知 |
| `platform_linux.go` | Linux：环境变量 + `gsettings` 代理、系统证书目录、`notify-send` |

### 通知系统

| 文件 | 职责 |
|------|------|
| `notifier.go` | `MultiNotifier` 多通道扇出，`notifyConfig` 读写 `~/.sushiro/notify.json` |
| `notifier_feishu.go` | 飞书 Webhook 卡片通知 |
| `notifier_telegram.go` | Telegram Bot API |
| `notifier_bark.go` | Bark iOS 推送 |
| `notifier_serverchan.go` | Server酱 |
| `notify.go` | `defaultString` 等小工具 |

### 功能模块

| 文件 | 职责 |
|------|------|
| `booking.go` | `cmdList`/`cmdCancel` CLI 命令，`onBookingSuccess` 成功后逻辑（状态/通知/日志） |
| `calendar.go` | `cmdCalendar` 终端日历网格 |
| `sniper.go` | 狙击模式：开放前 30 天精准抢号，50ms 高速轮询 |
| `history.go` | `history.jsonl` 追加（节流 30s），`cmdTrends` 趋势分析 |
| `recommend.go` | `cmdRecommend` 基于历史数据的时段推荐 |
| `insights.go` | Web/CLI 可复用的历史洞察：按门店/星期/时段统计开放概率、售罄速度与推荐 |
| `activity.go` | 主流程活动标记与信息收集跨进程锁，确保信息收集避让抢号/捕获/狙击 |
| `queue_trends.go` | 本地排队数据结构、到店预测推荐、过号趋势聚合、节假日分类、信息收集状态提示 |
| `sampling.go` | 后台信息收集配置、运行状态、定时 runner，仅记录历史不抢号 |
| `sampling_cli.go` | `sample` CLI：单次信息收集、前台信息收集、后台静默信息收集 start/stop/status |
| `update_check.go` | GitHub Latest Release 检查与版本比较 |
| `health.go` | 每 5 分钟验证 Token 有效性 |
| `state.go` | `State` JSON 读写，`logMessage`，`readInput` |
| `store.go` | `StoreRegistry` 门店昵称管理 `~/.sushiro/stores.json` |
| `diagnostics.go` | doctor 只读诊断、通知测试、本机网络/证书/端口/代理链路检查 |
| `maintenance.go` | repair-proxy / uninstall 的代理恢复和本地敏感数据清理 |
| `sniper_plan.go` | Web 狙击计划持久化、倒计时、尝试次数与状态摘要 |

### 资源与脚本

| 文件 | 职责 |
|------|------|
| `assets/sushiro.png` | 寿司郎官方 Logo PNG（base64 嵌入到 `web_static.go` 的 `logoBase64` 常量中） |
| `scripts/bundle-macos.sh` | Mac .app + DMG 桌面应用打包脚本 |
| `cloudflare/sushiro-cloud/` | Cloudflare Worker：GitHub OAuth、HMAC session、Turso secrets 和固定白名单查询 |
| `install/install.sh` | macOS/Linux 一键安装脚本 |
| `install/install.ps1` | Windows PowerShell 一键安装脚本 |

### CI/CD

| 文件 | 职责 |
|------|------|
| `.github/workflows/ci.yml` | 常规 CI：push/PR 运行测试、vet、gofmt、go mod tidy diff、安装脚本语法检查 |
| `.goreleaser.yml` | GoReleaser v2 配置：多平台编译 + Mac Universal Binary |
| `.github/workflows/release.yml` | GitHub Actions：tag 触发 → GoReleaser → Mac .app 打包 → 上传 Release |

---

## 数据文件路径

所有用户数据统一存放在 `~/.sushiro/` 目录：

```
~/.sushiro/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ryujoxys/sushiro-overdose](https://github.com/Ryujoxys/sushiro-overdose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
