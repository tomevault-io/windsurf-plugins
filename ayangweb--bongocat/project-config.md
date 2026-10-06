---
trigger: always_on
description: 适用于整个仓库；分析、修改或验证前先完整阅读本文件与更深层目录中的 `AGENTS.md`。
---

# BongoCat AI Working Agreement

适用于整个仓库；分析、修改或验证前先完整阅读本文件与更深层目录中的 `AGENTS.md`。

本文件只保留**长期有效的约束**。事实来源分工如下，冲突时指出并请求确认，不自行选择：

| 主题                         | 事实来源                          |
| ---------------------------- | --------------------------------- |
| 目标架构                     | `docs/technical-design.md`        |
| 单项决策背景与取舍           | `docs/adr/`                       |
| 依赖与工具校验               | `deny.toml`、`tools/`、`justfile` |

## 1. 项目事实

- Rust 2024 桌面应用：GPUI 设置 UI + Product/Live2D runtime。Windows 用 Raw Input、Win32、D3D11；macOS 用 CGEventTap、AppKit、Metal；共享 schema、资源与 fixture 保持平台无关。
- 首发平台是 Windows 10 1903+ 与 macOS 12+；Linux 只作首发后评估，不阻塞首发。
- Windows 只支持 `x86_64-pc-windows-msvc`，不得新增 i686 或 `aarch64-pc-windows-msvc` 的构建、测试或安装包；Windows on ARM 走 x64 仿真。
- “纯 Rust”指自有应用代码全部用 Rust；官方 Cubism Core 平台二进制是唯一允许的厂商 FFI 例外，业务逻辑不得放进 SDK bridge。
- Bundle ID 固定 `com.ayangweb.bongo-cat`。Development 与 Production 共用数据结构、使用不同存储根；开发构建不得读取、写入或锁住生产数据。
- 不引入 Tauri、WebView、Node.js、JavaScript 或第二套 UI framework。

## 2. 任务流程

### 2.1 实现决策阶梯（ADR-0030）

按顺序取第一个可行方案：现有行为已满足（YAGNI）→ 复用仓库已有 helper、contract、类型或模式 → Rust 标准库 → 操作系统原生能力 → workspace 已安装依赖 → 经调研的成熟第三方方案 → 最小自有代码。

- 第三方方案按 §6 审查版本、许可证、维护状态、平台支持、unsafe 面积和替换边界。
- 复用不得让第三方类型泄漏进项目公共 API；不复实现已有通用能力，不引入不可靠或不成比例的依赖。

### 2.2 开始前

1. 检查当前分支和 `git status`，保留全部用户修改；先 `git fetch` 把上游同步到最新（见 §10）。
2. 定位 TODO 中对应阶段、任务和退出门槛；读现有实现与测试，不从文档标题推断行为。
3. 明确最小交付范围、平台范围和验证方式；跨阶段门禁先完成前置 spike，否则明确报告阻塞。

### 2.3 实施

1. 遵循既有模块边界与命名，改动只覆盖请求和当前 TODO 项；先完成最小闭环再扩展，不同时铺开多个未验证子系统。
2. 平台差异只放在 platform adapter 或 `cfg` 模块，不扩散到 runtime、model、config、UI。
3. 新依赖说明用途、维护状态、许可证和替换边界。
4. 不做无关重构、批量格式化，不覆盖用户未提交修改，不动无关 lockfile、生成文件与资产 metadata。

### 2.4 完成前

1. 按 §9 跑与风险相称的格式化、静态检查、测试和平台 smoke test。
2. 复查正常、错误、重启和 shutdown 路径，以及依赖方向、平台类型泄漏、`unsafe` 范围和日志隐私。
3. 完成定义全部满足并有证据时才勾选 TODO checkbox；架构、行为协议或退出条件变化时同步 Technical Design、TODO 与 ADR。
4. 最终报告列出改动、验证命令与结果、未运行测试、已知风险、CHANGELOG 判断和下一项未完成任务。文档任务无需声称运行代码测试。

## 3. 阶段与数据版本

- 每次只推进一个最小闭环；新增能力先有自动化 contract 或 ADR，再进入实现。ADR-0011 允许已通过自动化 contract 且不依赖缺失外部证据的模块先进入正式实现。
- 门禁未过的能力不得宣称完成（Live2D、输入可靠性、双平台渲染等）；未取得 Cubism 书面授权、SDK 分发、实机输入/UI/GPU、签名、Windows 实机安装升级卸载与 soak 证据时，不得分发含受限 artifact 的安装包。
- 不批量迁移历史功能，不为目录美观预建空 crate，未经授权不删除历史源码和行为对照。
- `config.json` 与 window state 显式带 `schema_version: 1`，保留单一严格解析入口：非 1 明确拒绝，不静默转换。
- 不做旧 Tauri/Pinia 格式的兼容、探测、字段 alias、自动导入或目录 fallback，也不为开发期中间版本写迁移分支。
- 历史源码只保留在远端受保护分支 `pre-refactor-tauri`，仅作行为考古与模型兼容参考，不重新接入工作树或依赖图；上游参考实现是 [MMmmmoko/Bongo-Cat-Mver](https://github.com/MMmmmoko/Bongo-Cat-Mver)，见 `docs/migration/bongo-cat-mver-reference.md`。结论须由实际代码、配置或实机行为证明。

### 3.1 配置与数据兼容性

新增或修改配置项、持久化数据、状态结构时，前提是**已存在的数据仍能正常读取并正常启动**。新版本不得因为配置或数据结构变化导致旧数据读取失败、启动失败或行为异常。

- 新增字段必须兼容“旧数据里没有该字段”的情况：用 `#[serde(default)]`、`Option<T>` 或等价默认值，**不得**把新字段设为必填，否则旧文件解析失败并落入恢复流程甚至启动失败。
- 收紧取值范围、改变字段语义、枚举成员或 ID 生成规则前，先确认旧数据仍合法；确实不兼容时采用迁移方案（顺序、幂等、失败可回滚），并单独建立 ADR、设计、测试与发布门禁，不得直接改坏已有数据。
- 结构变化后同步更新 `Default`、`validate`、生成的 JSON Schema（`just schema`）、`shared/config/` 下的 fixture 与 manifest，并补一条“旧数据 → 新代码”读取的回归测试。
- 同一原则适用于 window state、日志、备份、model store 和更新 manifest 等所有持久格式。
- 边界不变：`schema_version` 非 1 仍明确拒绝；ADR-0054 的“最新有效备份 → 默认配置”是读取失败后的兜底，不是版本迁移。

## 4. 架构边界

### 4.1 GPUI

- 只负责设置、模型管理、快捷键、权限、更新和诊断 UI；不驱动 Live2D frame loop，不接入 GPUI renderer 私有接口。
- `Entity` 只保存表单草稿、选择、导航等临时视图状态；不持有 pressed state、动画状态、Cubism model 或主猫 GPU 资源。
- UI 通过强类型 command 请求 runtime，通过带 revision 的 snapshot 显示结果；executor 不做阻塞文件、模型解析或 GPU 工作，不持有 runtime 写锁。

### 4.2 Runtime

- Runtime 是配置、输入、动画和当前模型状态的唯一事实来源，由单一 owner 管理可变业务状态。
- Command、InputEvent、RuntimeSnapshot、RenderSnapshot 必须强类型；禁止 `set_value(path, any)`、弱类型 JSON 业务消息和字符串事件协议。
- 时间逻辑使用可注入的单调时钟，不得用墙上时间驱动动画或输入延迟。

### 4.3 Renderer 与 Overlay

- 主猫使用独立原生 overlay（Windows D3D11、macOS Metal），不嵌入 GPUI renderer；renderer 只消费不可变 RenderSnapshot，不读配置、不决定动作、不访问 GPUI Entity。
- Overlay 的窗口、GPU、frame source 必须有明确 owner 和析构顺序。
- Shutdown 顺序：阻止新 frame tick → 停止输入生产者 → 确认 frame source 退出 → 停止 runtime → flush 配置 → 停止音频并 join → 释放 renderer/GPU → 销毁 overlay → 关闭 GPUI。

### 4.4 平台模块

- 共享业务 crate 不导入 Win32、Objective-C、GPUI 或 GPU handle；平台 API 封装在 `bongocat-platform` 或明确的平台子模块，返回稳定的项目类型和 error code，不泄漏裸指针或平台消息结构。
- 主线程限定、COM apartment、run loop 和 callback 生命周期写入 wrapper 的安全不变量。
- 不靠 cfg fallback 让 Linux 编译，也禁止 `#[cfg(any(target_os = "macos", target_os = "windows"))]` 及其反向用法；共享桌面代码用直接 cfg、`test` 或单一平台条件，由 `tools/tests/test_supported_platform_cfg.py` 强制。

## 5. 输入可靠性

Issue #47 的“按下后无释放”必须从架构处理，不能只增加动画超时。

- Key/button down/up、设备连接/断开和 command 走可靠、有序队列；溢出可观测、计数并触发安全恢复，不得静默丢失边沿事件。
- 鼠标移动和手柄轴可合并为 latest value，但不得阻塞 key/button release；每个 pressed key 最终必须由 release、状态校正或 Reset 清除。
- Windows：Raw Input 是键鼠主路径，正确处理 scan code、E0/E1、左右修饰键和 `RI_KEY_BREAK`，用 `GetAsyncKeyState` 校正 pressed set；锁屏、睡眠、设备移除、输入桌面变化和服务重启必须 Reset；低级 hook 只作补充，不得成为 pressed state 的唯一来源；回归 PixPin `Ctrl+Alt+A`、Win+L、PrintScreen 和 UAC 返回。
- macOS：listen-only CGEventTap 加明确的 TCC 权限状态；tap timeout、disable、权限变化、session 变化必须可恢复，用 `CGEventSourceKeyState` 校正 pressed set；锁屏、睡眠、快速用户切换和 tap 重建必须 Reset。

## 6. Cubism、FFI 与依赖

- 使用官方 Cubism Core 平台二进制并记录版本、来源、hash、架构和许可证；artifact 必须来自已固定基线，升级、替换或新增时同步 provenance、目标 ABI 和模型验证。
- Raw binding 只在 sys/wrapper 边界，原始指针不得离开 safe wrapper；Rust owner 保证 Moc、Model、buffer、texture 和 renderer 的存活/析构顺序；模型切换使用 prepare → validate → commit，失败保留当前可用模型。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayangweb/BongoCat](https://github.com/ayangweb/BongoCat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
