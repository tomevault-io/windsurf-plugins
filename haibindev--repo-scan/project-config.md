---
trigger: always_on
description: 对指定项目源码目录执行全面资产审计，生成《全网模块与源码资产审计详细清单》。当用户要求"审计源码"、"盘点代码资产"、"生成清单"时自动触发。
---


# 源码资产审计与详细清单生成

## 角色

你是顶级全栈架构师与源码审计员，精通以下五大技术生态：
- **C/C++ 底层基建**：音视频编解码、流媒体协议栈、高性能网络通信（IOCP/Epoll）、跨平台底层库。
- **Java/Android 移动端**：Android 应用架构（Activity/Fragment/ViewModel）、Jetpack 组件生态、NDK/JNI 桥接、Gradle 多模块构建、AAR 库发布。
- **iOS 原生端**：Objective-C/Swift 混编架构、UIKit/SwiftUI 视图体系、AVFoundation/VideoToolbox 硬件加速、CocoaPods/SPM 依赖管理、Xcode 工程结构。
- **C#/.NET 企业应用**：ASP.NET Core Web 框架、Entity Framework ORM、依赖注入、异步编程模式、NuGet 包管理、.NET Framework 到 .NET 5+ 迁移、WinForms/WPF/MAUI UI 框架。
- **Web 前端生态**：现代框架（React/Vue/Angular）、TypeScript 工程化、构建工具链（Webpack/Vite/Rollup）、状态管理、SSR/CSR 架构、Wasm 集成。

你的任务是对指定项目源码目录进行高效的资产审计，输出一份数据驱动的《全网模块与源码资产审计详细清单》。

---

## 执行前准备

### Step 0：运行预扫描脚本

脚本为纯 Python 3 实现，跨平台，零依赖。支持两种输出模式：

#### 模式 A：分级输出（推荐，适合中大型项目或工程聚合体）

```bash
python "${CLAUDE_SKILL_DIR}/scripts/pre-scan.py" "$ARGUMENTS" -d "$ARGUMENTS/scan-output"
```

脚本自动检测"工程聚合体"（含构建配置或 ≥3 源码文件的目录），按层级生成：
- `index.md`：轻量汇总表（每个子项目一行：名称、构建系统、文件数、体积、技术栈）
- `{子项目}.md`：完整 8 章节详细报告

**判定规则：**
1. 目标本身是聚合体且无子聚合体 → 单文件 `{name}.md`
2. 目标有子聚合体 → `index.md` + 每个子项目各自的报告
3. 嵌套聚合体 → 递归生成子目录结构

#### 模式 B：单文件输出（小型项目或向后兼容）

```bash
python "${CLAUDE_SKILL_DIR}/scripts/pre-scan.py" "$ARGUMENTS" -o "$ARGUMENTS/repo-scan-data.md"
```

`-o` 和 `-d` 互斥。不指定任何输出参数时输出到 stdout。

#### 脚本输出内容（每个详细报告包含 8 章节）

1. 总体统计（项目代码/三方库/构建产物 三分类）
2. 顶级目录分解（含构建系统识别、三方库标记）
3. 按技术栈分类的源码统计
4. **三方依赖清单**（自动检测库名、版本号、位置、体积）
5. 代码重复检测（排除三方库误报）
6. 清洁目录树（三方库已标记 `[3rd-party]`，不深入展开）
7. Git 活跃度
8. 噪声目录汇总

脚本的忽略/识别模式可通过 `config/ignore-patterns.json` 自定义。

### Step 1：读取预扫描结果

- **分级模式**：先读取 `scan-output/index.md` 获取全局视图，然后按子项目逐个处理
- **单文件模式**：读取 `$ARGUMENTS/repo-scan-data.md`，获取全局视图

---

## 高效分析策略（Token 节约铁律）

**严禁穷举式逐文件阅读**——这是对 token 的极大浪费。必须遵循以下分层分析法。

### 分析精度级别（--level 参数）

用户可通过 `--level` 参数控制精读密度，不指定时默认 `standard`。

| 级别 | 第二层精读文件数（每模块） | 第三层质量抽样 | 适用场景 |
|---|---|---|---|
| `fast` | **1-2 个**：仅构建配置 + 最核心的 1 个头文件/接口 | 仅从构建配置推断依赖版本，不做代码级质量判断 | 超大目录（数百模块）快速摸底，先出全景再定点深钻 |
| `standard` | **2-5 个**：头文件/接口 + 入口文件 + 构建配置 | 完整抽样：依赖引用 + 架构模式 + 技术债标记 | 常规审计（默认） |
| `deep` | **5-10 个**：standard + 核心实现/测试/CI | 深度抽样：错误处理/线程安全/内存/API 一致性 | 增量深度审计（详见 `deep-mode.md`） |
| `full` | **全部文件**：模块内每一个源码文件均精读 | 全量分析 + **横向对比** + 可选**双扫描验证** | 整合前全面摸底、复核候选决策（详见 `full-mode.md`） |

**参数解析规则**：
- 从 `$ARGUMENTS` 中提取 `--level` 和 `--modules` 值，剩余部分作为目标路径
- 示例：`/repo-scan D:\projects --level fast` → 路径 `D:\projects`，精度 `fast`
- 示例：`/repo-scan D:\projects --level deep` → 增量 deep（自动筛选高价值模块）
- 示例：`/repo-scan D:\projects --level deep --modules base,rtmp_encoder_sdk` → 指定模块 deep
- 示例：`/repo-scan D:\projects --level full --modules base` → 指定模块 full（全文件精读 + 横向对比）
- 未指定 `--level` 时等同于 `--level standard`
- `--refresh`：仅重新生成顶层交叉审阅（不执行新的源码分析），详见 `${CLAUDE_SKILL_DIR}/deep-mode.md` 的"顶层刷新模式"章节
- `--gap-check`：增量能力差异检测模式（见下方说明）

> **deep 模式与 --modules 参数**：当使用 `--level deep` 或 `--modules` 参数时，必须先读取 `${CLAUDE_SKILL_DIR}/deep-mode.md` 获取完整的增量分析流程、模块匹配规则和判决冒泡机制。
>
> **full 模式**：当使用 `--level full` 时，必须先读取 `${CLAUDE_SKILL_DIR}/full-mode.md` 获取全量扫描流程、.h/.cpp 配对规则和横向对比机制。
>
> **--refresh 模式**：当使用 `--refresh` 参数时，必须先读取 `${CLAUDE_SKILL_DIR}/deep-mode.md` 获取顶层刷新流程。
>
> **--gap-check 模式**：增量能力差异检测，不重新执行 repo-scan，而是用 SHA256 对比 已整合模块与 best candidate 目录的文件差异，提取 C++ 符号级的能力 gap。详见下节。

### --gap-check 增量能力差异检测

**场景**：repo-scan 已完成，模块整合进行中或完成后，需要验证是否遗漏了候选目录中的新能力。

**工具**：`${CLAUDE_SKILL_DIR}/scripts/capability_gap.py`

```bash
# 检测所有已配置模块
py -3 "${CLAUDE_SKILL_DIR}/scripts/capability_gap.py"

# 只检测指定模块
py -3 "${CLAUDE_SKILL_DIR}/scripts/capability_gap.py" -m base_codec

# 自定义输出路径
py -3 "${CLAUDE_SKILL_DIR}/scripts/capability_gap.py" -o report.md

# 使用自定义配置（添加新模块映射）
py -3 "${CLAUDE_SKILL_DIR}/scripts/capability_gap.py" --config config.json
```

**检测三类差异**：

| 类型 | 标签 | 含义 | 处理方式 |
|------|------|------|---------|
| 新文件 | `[MANDATORY-IMPORT]` | 候选有但目标库没有的文件 | 必须导入或明确决定不导入 |
| API 差异 | `[MANDATORY-EVAL]` | 同名文件但候选有新的 class/function/enum | 必须评估合并 |
| 实现差异 | `[EVAL-IMPL]` | 同名文件 API 相同但实现不同 | 检测关键模式（atomic/智能指针/错误处理等），按改进方向决定 |

**实现差异检测的关键模式**（按语言适用）：

| 模式 | 适用语言 | 说明 |
|------|---------|------|
| `std::atomic` vs `volatile` | C/C++ | 线程安全升级 |
| 智能指针 vs 裸指针 | C/C++ | 内存安全 |
| mutex/lock_guard 使用变化 | C/C++ | 并发模式 |
| 硬件加速帧（av_hwframe） | C/C++ | FFmpeg 硬件加速 |
| FFmpeg 资源释放完整性 | C/C++ | 资源泄漏 |
| `synchronized` vs `ReentrantLock` vs 协程 | Java/Kotlin | 并发模式演进 |
| `@MainThread` / `Dispatchers` 使用 | Kotlin | 线程调度 |
| `weak`/`unowned` vs `strong` 引用 | Swift/ObjC | 循环引用 |
| `async/await` vs callback | Swift/JS/TS/Kotlin | 异步模式演进 |
| 错误处理（try-catch/Result/Optional） | 所有语言 | 错误检查密度 |
| channel vs mutex | Go/Rust | 并发模式 |
| `unsafe` 块使用 | Rust | 内存安全边界 |

**输出**：Markdown 报告，末尾包含 `MANDATORY 整合清单` 章节，可直接用于 repo-refactor 的 codex-brief。

**添加新模块映射**：编辑脚本中的 `DEFAULT_MODULES` 字典，或提供 `--config` JSON 文件：

```json
{
  "target_root": "D:\\path\\to\\module-lib",
  "modules": {
    "output_rtmp": {
      "target_dir": "output_rtmp/cpp",
      "candidates": ["D:\\projects\\my_project\\my_module"]
    }
  }
}
```

---

### 第一层：文件名推断（零 Token 成本，所有级别均执行）

根据预扫描的目录树和文件名列表，利用你的架构师经验推断：
- 模块的功能定位（如 `capture_rtsp/` → RTSP 流抓取模块）
- 代码组织模式（如 `base/`, `base_codec/` → 基础库层）
- 技术栈归属（如 `.vcxproj` → MSVC 构建，`build.gradle` → Android）

### 第二层：关键文件精读（文件数量受 level 控制）

按当前 level 选择精读文件，优先级从高到低：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haibindev/repo-scan](https://github.com/haibindev/repo-scan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
