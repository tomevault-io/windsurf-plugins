---
trigger: always_on
description: 本文件为在本仓库工作的 AI 编码助手提供上下文。修改代码前请先阅读本文档与 [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)。
---

# AGENTS.md

本文件为在本仓库工作的 AI 编码助手提供上下文。修改代码前请先阅读本文档与 [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)。

## 项目状态

| 项 | 说明 |
|----|------|
| **类型** | 个人项目（Skymly 工作区） |
| **远端** | https://github.com/Skymly/DesignPatterns（**公开**）；文件夹名 `DesignPatterns` = 仓库名 |
| **许可证** | [MIT](LICENSE) |
| **阶段** | **早期预览**：公共 API、生成器产出与 `DP###` 诊断**尚未稳定**（见 [README.md](README.md)） |
| **NuGet** | 元包 **`Skymly.DesignPatterns`**；当前 **`0.2.4-preview1`**；`release.yml` + Nuke `Publish` → nuget.org + GitHub Packages |
| **Sibling 仓库** | [DesignPatterns.Samples](https://github.com/Skymly/DesignPatterns.Samples)、[DesignPatterns.Docs](https://github.com/Skymly/DesignPatterns.Docs) — 本地并列路径 `C:\Code\Skymly\DesignPatterns\DesignPatterns.Samples`、`C:\Code\Skymly\DesignPatterns\DesignPatterns.Docs` |

## 项目是什么

**DesignPatterns** 是一个以**技术探索**为目的的 .NET 设计模式工具库：发挥当前库的最大技术潜能，**即使与现有其他项目（MediatR / Polly / `Microsoft.Extensions.*` / Stateless 等）能力重叠也无所谓**——重叠不是拒绝实现的理由，能否在编译期胶水 + 运行时 primitives 的组合上做出有技术价值的探索才是判断标准。

- **运行时**：轻量、可组合的 primitives（责任链、策略、工厂注册表、Singleton 特性等），组合优于继承。
- **编译期**：`DesignPatterns.SourceGenerators` 源生成器；`DesignPatterns.Analyzers` 诊断；`DesignPatterns.CodeFixes` CodeFix。
- **语言**：**C# 优先**；后续再考虑 .NET 生态其他语言（F# / VB 等待评估）。
- **模式范围**：**包含 GoF 但不局限于 GoF**——并发模式、反应式模式、函数式模式、分布式模式等只要能用「primitive + 编译期胶水」表达且具备技术探索价值，均可纳入。

仍须遵守的硬约束（与探索方针并存，非自我设限而是工程底线）：primitives 而非厚重基类体系；Core 不引用 MSDI；异步一等；显式失败优先 `TryResolve` 与明确异常。

---

## 仓库结构

```
DesignPatterns.slnx
├── DesignPatterns/                              # 运行时核心（netstandard2.0 + net8.0）
├── DesignPatterns.Diagnostics/                  # DiagnosticIds 常量（DP001–DP055）
├── DesignPatterns.SourceGenerators/             # 增量源生成器
├── DesignPatterns.Analyzers/                    # DP006、DP023、DP024、DP025、DP033、DP036 Analyzer
├── DesignPatterns.CodeFixes/                    # CodeFixProvider
├── DesignPatterns.Extensions.DependencyInjection/  # MSDI 扩展 + DI 生成器 targets
├── DesignPatterns.Extensions.Autofac/              # Autofac 扩展 + Autofac 生成器 targets
├── DesignPatterns.Package/                      # NuGet 元包（PackageId=Skymly.DesignPatterns）
├── tests/                                       # 单元 / 生成器 Verify / Analyzer / DI
├── docs/                                        # 维护者文档（DOCUMENTATION、DEVELOPMENT、ROADMAP、adr/、design/）
├── .github/                                     # Issue/PR 模板、CI
└── AGENTS.md
```

**不**经 NuGet 元包实现：自定义 Roslyn `CompletionProvider`（IDE 宿主不加载项目引用的 Provider；成员补全已由 `*Keys` 的 `public const string` 与 DP025 字面量键校验覆盖）。独立 VSIX/Rider 插件暂不排期（见 [ROADMAP](docs/ROADMAP.md) F1）。

---

## 跨模块 PR / Issue 边界

与 [`DesignPatterns.slnx`](DesignPatterns.slnx) 一致；**每个模块单独 Issue + PR**（勿在同一 PR 混合多个模块）：

| 模块 | 范围 |
|------|------|
| **Runtime** | `DesignPatterns/` |
| **Diagnostics** | `DesignPatterns.Diagnostics/` |
| **SourceGenerators** | `DesignPatterns.SourceGenerators/` |
| **Analyzers** | `DesignPatterns.Analyzers/` + `DesignPatterns.CodeFixes/` |
| **DependencyInjection** | `DesignPatterns.Extensions.DependencyInjection/`、`build/*.targets` |
| **Autofac** | `DesignPatterns.Extensions.Autofac/`、`build/*.targets` |
| **Package** | `DesignPatterns.Package/` |
| **Docs** | **仅**本仓 `docs/`（ADR、Design Doc、ROADMAP、维护者笔记、`docs/agents/`）。用户文档站工作在 [DesignPatterns.Docs](https://github.com/Skymly/DesignPatterns.Docs)（独立 Issue + PR） |
| **Repository (root README)** | 根 `README.md`、`CONTRIBUTING.md` |
| **Repository** | `.github/`、`DesignPatterns.slnx`、`global.json`、`Directory.Build.props`、`build/` |

模式域（Behavioral / Creational / Structural）变更仍须落在上表某一模块内；跨模式且跨模块时拆多个 Issue → PR。

---

## 常用命令

与 CI 一致（Nuke）：

```powershell
# 编译 + 测试（默认本地 Debug；CI 用 Release）
./build.ps1 --target Ci --configuration Release

# 打包 + 校验 nupkg → artifacts/package/
./build.ps1 --target CiPack --configuration Release
```

等价：

```powershell
dotnet run --project build/_build.csproj -- --target Ci --configuration Release
dotnet run --project build/_build.csproj -- --target CiPack --configuration Release
```

传统 dotnet（仍可用，非 CI 权威路径）：

```powershell
dotnet build DesignPatterns.slnx -c Release
dotnet test DesignPatterns.slnx -c Release
```

启用生成器 DI 路径：引用 `DesignPatterns.Extensions.DependencyInjection`（自动 Import `build/DesignPatterns.Extensions.DependencyInjection.targets`）。

启用生成器 Autofac 路径：引用 `DesignPatterns.Extensions.Autofac`（自动 Import `build/DesignPatterns.Extensions.Autofac.targets`）。

---

## 已实现的模式（摘要）

| 模式 | 特性 / API | 生成器 |
|------|------------|--------|
| Singleton | `[GenerateSingleton]` | `GenerateSingletonGenerator` |
| Factory Registry | `IFactoryRegistry`、`[RegisterFactory]` | `RegisterFactoryGenerator` |
| Strategy | `[RegisterStrategy]` | `RegisterStrategyGenerator` |
| Chain | `IHandler<T>`、`HandlerPipeline` | `HandlerOrderGenerator` |
| Composite | `[CompositePart]` | `CompositePartGenerator` |
| Decorator | `[Decorator]` | `DecoratorGenerator` |
| Event Aggregator | `IEventAggregator`、`[RegisterEventHandler]` | `RegisterEventHandlerGenerator` |
| Command Router | `ICommandRouter`、`IStreamCommandHandler`、`[RegisterCommandHandler]` | `RegisterCommandHandlerGenerator` |
| State（M1–M2） | `ITransitionTable`、`[StateMachine]`、`[Transition]` | `StateTransitionGenerator` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skymly/DesignPatterns](https://github.com/Skymly/DesignPatterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
