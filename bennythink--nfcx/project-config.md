---
trigger: always_on
description: 本文件适用于整个 NFCX 仓库。它既是项目背景说明，也是后续开发代理必须遵守的架构和实现约束。
---

# NFCX 项目开发指南

本文件适用于整个 NFCX 仓库。它既是项目背景说明，也是后续开发代理必须遵守的架构和实现约束。

## 项目背景

NFCX 是一个面向 Windows、macOS 和 Linux 的桌面 NFC 工具，目标是提供类似 MifareOne Tool 的图形化操作体验，同时避免只支持 Windows、只支持单一串口转换器或要求用户手工安装大量命令行工具。

项目最初使用 PN532 + FT232RL，通过 PN532 UART/HSU 模式完成验证。`scripts/` 中的 Python 程序已经证明当前硬件链路可以：

- 检测 PN532 和卡片；
- 读取 MIFARE Classic 1K；
- 将卡片保存为原始 dump；
- 将 dump 写回卡片；
- 检查 BCC、访问控制位并进行写后校验。

这些脚本是协议验证和硬件回归工具，不是正式桌面应用的运行时依赖。不要简单地把脚本里散落的 PN532 十六进制帧逐行翻译成 Go。

NFCX 只应用于用户拥有或明确获准测试的卡片和系统。

## 产品方向

NFCX 的定位是“小而完整的桌面工具”，不是通用 NFC 研究框架。优先保证：

1. 三个平台上的一致 GUI；
2. PN532 UART + FT232RL 开箱可用；
3. 普通读写稳定且不会轻易写坏卡片；
4. libnfc 支持的其他读卡器能够逐步接入；
5. 复用成熟的 MIFARE Classic 密钥恢复工具，不在 Go 中重新实现破解算法。

第一阶段以 MIFARE Classic 1K 为重点。除非当前任务明确要求，否则不要为了支持更多卡型而扩大实现范围。

## 固定技术栈

- Go：**1.25.1**；`go.mod` 必须以此版本为项目基线。
- 桌面框架：Wails v2。
- 前端：Vanilla TypeScript、HTML、CSS。没有明确收益时不要引入 React、Vue 或大型组件库。
- NFC 硬件抽象：libnfc。
- Go/libnfc 集成：CGO + 一个很薄的项目内 C shim。
- 破解引擎：以子进程方式调用 `mfoc`、`mfcuk` 等原生命令行程序。
- 串口：由 libnfc 对应驱动管理。正式主路径不要再单独维护一套 Go PN532 UART 协议栈。
- 测试：Go 标准测试、mock reader、固定响应/dump fixtures；真实硬件测试必须显式启用。

不采用 Electron。除非用户明确改变技术路线，否则不要迁移到 Tauri，也不要让 Rust 成为构建依赖。

## 总体架构

```text
Wails GUI
    |
Go application services
    |-- DeviceManager
    |-- CardService
    |-- DumpService
    |-- KeyService
    `-- AttackManager
          |
          |-----------------------------|
          |                             |
    in-process libnfc              external processes
    through CGO shim               mfoc / mfcuk / others
          |                             |
          `-------------+---------------'
                        |
                    NFC reader
```

必须保持下面三层边界：

1. GUI 只调用 Go application service，不直接接触 CGO、libnfc 或 `os/exec`。
2. 普通 NFC 操作通过项目内的 `Reader` 接口完成。
3. 破解算法通过 `AttackEngine` 接口运行外部程序。

## 哪些功能链接 libnfc

以下功能在 NFCX 主进程内通过 CGO 链接 libnfc，并通过 C shim 调用其 API：

- 枚举 libnfc 设备和连接字符串；
- 打开、初始化和关闭读卡器；
- 检测读卡器状态；
- 选择 ISO/IEC 14443A 卡片；
- 获取 UID、ATQA、SAK 等卡片信息；
- MIFARE Classic Key A/Key B 认证；
- 读取单个 block；
- 写入单个 block；
- 已知密钥的逐扇区扫描；
- 普通卡整卡 dump；
- 普通卡 dump 恢复；
- 写入后的逐块回读验证；
- CUID 等允许使用普通写命令修改 block 0 的操作，但必须经过危险操作保护。

上层不应直接使用 `nfc_context`、`nfc_device`、`nfc_target` 或 libnfc 的 union/struct。它们只能出现在 `internal/nfc/libnfc` 或等价的封装目录中。

推荐的 Go 接口形态：

```go
type Reader interface {
	Open(ctx context.Context, connString string) error
	Close() error
	CardInfo(ctx context.Context) (CardInfo, error)
	Authenticate(ctx context.Context, block byte, keyType KeyType, key Key) error
	ReadBlock(ctx context.Context, block byte) ([16]byte, error)
	WriteBlock(ctx context.Context, block byte, data [16]byte) error
}
```

C shim 只暴露 NFCX 实际需要的窄接口，并将 libnfc 错误转换为稳定的 NFCX 错误码。不要在 Wails binding 或业务层直接写 `import "C"`。

## 哪些功能调用外部程序

以下功能不链接进 NFCX 主进程，而是通过 `os/exec` 启动对应平台的外部二进制：

- `mfoc`：存在至少一个已知密钥时执行 Nested 密钥恢复；
- `mfcuk`：尝试通过 Darkside 流程恢复第一把密钥；
- `mfoc-hardnested` 或选定的等价工具：未来的 Hardnested 支持；
- `nfc-mfsetuid` 或 NFCX 维护的兼容版本：需要特殊后门序列的 Gen1A/UID 卡操作；
- 其他专用密码分析工具：只有在定义了明确适配器后才允许加入。

Windows 上这些文件通常带 `.exe`；macOS 和 Linux 使用对应的本地可执行文件。代码和文档中不要把外部引擎统称为 Windows EXE。

不要通过 shell 拼接命令。必须使用 `exec.CommandContext` 和参数数组，避免路径空格、转义差异和命令注入：

```go
cmd := exec.CommandContext(ctx, executablePath, args...)
```

外部程序必须通过统一接口接入：

```go
type AttackEngine interface {
	Name() string
	Available(ctx context.Context) error
	Run(ctx context.Context, request AttackRequest, emit func(AttackEvent)) (AttackResult, error)
}
```

不要让 GUI 了解 `mfoc` 或 `mfcuk` 的具体参数格式。

## libnfc 和外部引擎的设备所有权

一个读卡器同一时间只能由一个操作持有。普通扫描、读写和外部破解任务共享一个全局 `DeviceManager` 与设备锁。

启动外部破解程序时必须按顺序执行：

1. 停止 GUI 后台轮询；
2. 关闭主进程内的 libnfc `nfc_device`；
3. 使用选中的准确 connstring 启动外部程序；
4. 实时转发 stdout/stderr，并支持取消；
5. 等待进程退出并校验结果文件；
6. 无论成功、失败还是取消，都清理临时状态；
7. 重新打开读卡器；
8. 使用主进程内的 libnfc 验证恢复出的密钥。

不得让 in-process libnfc 和外部工具同时访问同一设备。

外部进程应继承同一设备选择，例如通过进程环境设置 `LIBNFC_DEVICE=<connstring>`。不要修改用户的系统级 `/etc/nfc` 或全局 libnfc 配置。

## 外部程序输出处理

所有外部任务都要保留原始 stdout/stderr 供 GUI 日志查看，但不要把自然语言输出当成唯一成功依据。

- 首先检查退出码；
- 检查预期输出文件是否存在；
- 验证 dump 大小、结构、sector trailer 和访问控制位；
- `mfoc` 的最终密钥优先从输出 dump 中读取；
- 只有 `mfcuk` 等没有结构化结果的工具才解析少量稳定文本；
- 如果上游输出不稳定，优先维护一个很小的 fork，增加 `NFCX_RESULT ...` 机器可读行，而不是复制整个算法到 Go。

外部任务的取消必须真正终止进程，并在需要时处理它创建的子进程。不要只在 GUI 上隐藏进度窗口。

## libnfc 构建和分发

主程序的 CGO 封装和外部破解工具必须尽量使用同一固定版本、同一驱动配置的 libnfc。不要在 CI 中不加锁定地追踪上游最新提交。

优先启用的驱动：

- `pn532_uart`：首要目标，覆盖 PN532 + FT232RL；
- `pn53x_usb`；
- `acr122_usb`；
- `acr122_pcsc`/`pcsc`：按平台构建能力启用；
- 后续驱动只有经过测试才在 GUI 中宣称支持。

“libnfc 能识别”不代表“支持所有破解功能”。设备能力至少区分：

- 可枚举；
- 可寻卡；
- 可进行 MIFARE Classic 普通读写；
- 支持 Nested；
- 支持 Darkside；
- 支持 Hardnested；
- 支持特殊 UID/后门命令。

安装包预计包含：

```text
runtime/
  windows-amd64/
    libnfc.dll
    required runtime DLLs
    mfoc.exe
    mfcuk.exe
  darwin-arm64/
    libnfc.dylib
    mfoc
    mfcuk
  linux-amd64/
    libnfc.so
    mfoc
    mfcuk
```

使用每个平台的原生 CI runner 构建 CGO、Wails、libnfc 和外部引擎。不要默认依赖从 Linux 交叉编译所有平台。

发布优先级：

1. macOS arm64；
2. Windows amd64；
3. Linux amd64；
4. 经过需求验证后再增加 macOS amd64、Windows arm64、Linux arm64。

macOS 发布时，嵌套的 dylib 和外部可执行文件必须先签名，再签名并公证整个应用。Windows 发布不要使用 UPX 压缩，以降低安全软件误报概率。Linux 优先提供 AppImage 和 tar.gz。

## GUI 功能范围

第一版 GUI 至少包括：

- 设备列表、刷新、手动 connstring 和连接状态；
- 卡片放入/移除状态；
- UID、ATQA、SAK 和识别出的卡型；
- 扇区/block 十六进制查看；
- Key A/Key B 输入、导入和逐扇区状态；
- 默认密钥字典扫描；
- 整卡读取并保存 `.bin`/`.mfd`；
- 加载 dump；
- 单块、单扇区和整卡写入；
- 写后验证及失败 block 标记；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BennyThink/NFCX](https://github.com/BennyThink/NFCX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
