---
trigger: always_on
description: Automated code audit using static syntax checking tools. Invoke when user requests code security audit, code quality review, static analysis, syntax checking, or compliance verification for any codebase or project.
---


# 代码审计技能 (Code Auditor)

教 LLM 真正理解和运用静态语法纠错工具进行代码审计，而不是只会调用预设脚本。

## 核心思想

```
┌─────────────────────────────────────────────────────────────┐
│  真正懂审计 = 理解工具原理 + 灵活构建命令 + 智能解读输出  │
│  而不是：找到脚本 → 执行 → 粘贴结果                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 工作流程总览

1. **发现目标** — 搜索并定位要审计的代码文件/目录
2. **克隆仓库** — 如果是 GitHub 仓库链接，先 `git clone` 到本地再审计
3. **语言检测** — 通过扩展名、配置文件识别编程语言
4. **询问审计策略** — 主动问用户要什么深度（语法检查 / 标准审计 / 深度安全），不要默认假设
5. **工具认知** — 根据选定的策略深度，选择对应类型的工具（编译器/linter/格式化器/安全扫描）
6. **构建命令** — 根据项目结构、审计策略和环境动态构造审计命令
7. **执行审计** — 依次运行各工具的审计命令，捕获 stdout/stderr
8. **解读结果** — 智能解读工具输出，区分致命错误、潜在问题和风格建议
9. **按需读码** — 仅当工具报告语法/编译错误时，读取报错文件的源码进行复核
10. **报告呈现** — 生成结构化审计报告

---

## 步骤一：发现目标文件

根据用户描述或项目上下文，自动搜索需要审计的文件：

- 如果用户指定了文件或目录路径，直接使用
- 如果用户没有指定，根据上下文推测目标
- 使用 `Glob` 或 `LS` 等工具搜索项目文件

通过项目根目录的配置文件推断语言生态：

| 配置文件 | 推断语言 | 典型项目结构 |
|---------|---------|-------------|
| `CMakeLists.txt` / `Makefile` | C/C++ | `src/`, `include/`, `lib/` |
| `pom.xml` / `build.gradle` | Java | `src/main/java/`, `src/test/java/` |
| `Cargo.toml` | Rust | `src/main.rs`, `src/lib.rs` |
| `go.mod` | Go | 平铺或多级 package |
| `package.json` | JS/TS | `src/`, `node_modules/` |
| `pyproject.toml` / `requirements.txt` | Python | `src/`, `tests/` |
| `*.sln` / `*.csproj` | C# | 多项目 Solution |
| `mix.exs` | Elixir | `lib/`, `test/` |
| `pubspec.yaml` | Dart/Flutter | `lib/`, `test/` |
| `Package.swift` | Swift | `Sources/`, `Tests/` |

> **关键理解**：配置文件不仅告诉你用什么语言，还告诉你项目的**模块结构和构建方式**，这直接决定了你应该用什么范围的命令（单文件检查 vs 全项目检查）。

---

## 步骤二：语言检测规则

### 文件扩展名识别

```
.c/.h/.cpp/.hpp/.cc/.cxx → C/C++
.java                       → Java
.kt/.kts                    → Kotlin
.cs                         → C#
.py/.pyw                    → Python
.js/.jsx/.mjs/.cjs          → JavaScript
.ts/.tsx                    → TypeScript
.go                         → Go
.rs                         → Rust
.rb                         → Ruby
.php/.phtml                 → PHP
.sh/.bash/.zsh              → Shell
.swift                      → Swift
.dart                       → Dart/Flutter
（以及其他常见扩展名）
```

### Shebang 识别（无扩展名文件）

```
#!/usr/bin/env python    → Python
#!/bin/bash              → Shell
#!/usr/bin/ruby          → Ruby
#!/usr/bin/perl          → Perl
#!/usr/bin/php           → PHP
```

---

## 步骤三：审计策略确认（问用户，别猜）

**核心原则**：不要默认假设用户想要多深的审计。每次接到审计任务，先问用户再动手。

LLM 在完成语言检测后，应主动向用户提出以下问题来确认审计策略：

### 标准提问模板

```markdown
已识别到项目语言为 **<语言>**，审计目标为 **<文件/目录路径>**。

请选择审计深度（三选一）：

❶ **快速语法检查** — 只检查语法错误，最快完成
   适用：提交前快速验证、CI 预检查、修改后自查
   工具：仅编译器语法检查（如 gcc -fsyntax-only / node --check）
   耗时：数秒

❷ **标准质量审计** — 语法 + 代码质量 + 风格
   适用：常规代码审查、MR/PR 审查、定期质量检查
   工具：编译器 ⚡ + Linter 🔍（如 cppcheck / eslint / ruff）
   耗时：数十秒到数分钟

❸ **深度安全审计** — 语法 + 质量 + 安全专项扫描
   适用：上线前安全检查、第三方代码审计、安全合规
   工具：编译器 ⚡ + Linter 🔍 + 安全专项 🛡️（如 shellcheck / bandit / cppcheck --enable=warning）
   耗时：数分钟到十分钟
```

> **LLM 注意**：你要根据项目的实际情况动态填充 `<语言>` 和 `<路径>`，让用户清楚你在审计什么。

### 用户的回答可能不精确

用户可能不会直接说"我选二"，而是说自然语言。LLM 应能理解意图并映射到策略：

| 用户说 | 映射到策略 | 理由 |
|-------|-----------|------|
| "就看看有没有语法错误" | ❶ 快速语法检查 | 用户明确只要语法 |
| "帮我 review 一下代码质量" | ❷ 标准质量审计 | review 通常含质量检查 |
| "马上要上线了帮我扫一遍" | ❸ 深度安全审计 | 上线前需要安全把关 |
| "随便看看" / "你看着办" | ❷ 标准质量审计 | 默认中等深度 |
| "检查有没有安全漏洞" | ❸ 深度安全审计 | 明确要安全扫描 |

### 三层策略模板详解

#### ❶ 快速语法检查

| 维度 | 说明 |
|------|------|
| **目的** | 最快速度验证代码是否能通过编译/解析 |
| **工具** | 仅用编译器语法检查命令，不用 linter |
| **检查内容** | 语法错误、缺失符号、类型不匹配 |
| **不检查** | 代码风格、潜在 bug、安全漏洞 |
| **耗时** | 数秒 |
| **适用场景** | 改了一行代码想确认没写错、CI 快速门禁 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q file.py     # 仅语法检查
```
（不执行 pyflakes / ruff / flake8）

#### ❷ 标准质量审计

| 维度 | 说明 |
|------|------|
| **目的** | 平衡速度与覆盖面，常规代码审查 |
| **工具** | 编译器语法检查 + 主要 Linter |
| **检查内容** | 语法错误 + 潜在 bug + 反模式 + 代码风格 |
| **不检查** | 专项安全漏洞（如 SQL 注入、XSS） |
| **耗时** | 数十秒到数分钟 |
| **适用场景** | MR/PR 审查、定期代码巡检、日常质量把控 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q .           # 语法检查
ruff check .                        # 全面质量检查（替代 pyflakes + flake8）
```
（不执行专项安全扫描）

#### ❸ 深度安全审计

| 维度 | 说明 |
|------|------|
| **目的** | 最全面的检查，不遗漏任何潜在风险 |
| **工具** | 编译器 + Linter + 安全专项工具 + 最严格参数 |
| **检查内容** | 语法错误 + 潜在 bug + 代码风格 + **安全漏洞（注入、XSS、权限、密钥泄露）** |
| **耗时** | 数分钟到十分钟 |
| **适用场景** | 上线前安全检查、第三方代码审计、安全合规审查、高危项目 |

**典型命令组合**（以 Python 为例）：
```bash
python -m compileall -q .           # 语法检查
ruff check .                        # 质量 + 安全规则（ruff 包含安全规则）
flake8 .                            # 风格补充
```
（对 Shell 脚本还会加 shellcheck，对 JS 加 eslint-plugin-security 等）

---

## 步骤四：工具理解（核心部分）

**这是整个技能的灵魂**：LLM 必须理解每类工具的**检查维度**、**适用场景**和**输出风格**，才能灵活运用而不是死记命令。

### 工具类型总览

所有静态审计工具按功能分为三大类：

| 类型 | 本质 | 检查什么 | 速度 | 误报率 |
|------|------|---------|------|-------|
| **编译器语法检查** | 模拟编译，只解析语法树 | 语法错误、类型不匹配、未定义引用 | 最快 | 极低（都是真错误） |
| **Linter** | 静态分析 AST/CFG | 潜在 bug、反模式、安全漏洞、代码坏味 | 中等 | 中等（部分规则较主观） |
| **格式化器检查** | 解析→格式化→对比 | 缩进、空格、换行、命名风格 | 最快 | 极低（风格约定而已） |

> **理解这一点很重要**：编译器的"错误"是**确定性的**——代码就是错的。Linter 的"警告"是**概率性的**——可能有问题，需要人工判断。格式化器的"问题"是**约定性的**——不影响逻辑，只影响可读性。

### 如何为每种语言选择工具

LLM 应该根据**步骤三确定的策略 + 以下三个维度**综合决策：

```
维度1：项目范围

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snake-aabb-wtf/code-auditor-skill](https://github.com/snake-aabb-wtf/code-auditor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
