---
trigger: always_on
description: 本文件为在本仓库工作的 AI 编码助手提供上下文。修改代码前请先阅读本文档。文档约定见 [docs/DOCUMENTATION.md](docs/DOCUMENTATION.md)。
---

# Observables — AI 代理说明

本文件为在本仓库工作的 AI 编码助手提供上下文。修改代码前请先阅读本文档。文档约定见 [docs/DOCUMENTATION.md](docs/DOCUMENTATION.md)。

## 项目状态

- **类型**：个人项目（Skymly 工作区）
- **远端**：https://github.com/Skymly/Observables（私有）；文件夹名 `Observables` = 仓库名；同步状态以 `git status` 为准
- **阶段**：**Events**、**RestAPI**、**SignalR**、**Mqtt**、**WebSocket**、**Grpc**、**Sse**、**Nats**、**Postgres**、**Redis** 已实现（运行时 + 双路生成器 + 测试）；共享层另含 `Observables.CodeFixes` 与 `Observables.Analyzers`；**nuget.org 已发** `0.1.9`（**20 包**，含 Redis）；Nuke `PackVerify` + `eng/nuget-smoke` 覆盖 manifest 包清单
- **下一里程碑**：post-1.0 维护期（M1–M7 全部完成，`0.1.9` = 第十域 Redis 已发）；待定项见 [`docs/ROADMAP.md`](docs/ROADMAP.md) 末尾
- **路线图**：里程碑与发版顺序见 [`docs/ROADMAP.md`](docs/ROADMAP.md)（M1 ✅ … M7 ✅，0.1.7 = Postgres，0.1.9 = Redis）
- **结构约定**：下文「仓库结构」与命名约定为权威；**工程治理**（包管理、警告、诊断、版本来源）见下文同名章节

## 目标

实现一组 **Roslyn 源生成器**，将多种事件与 IO 边界桥接到反应式 API。

---

## 命名约定（权威）

### 两层名称：解决方案项目 vs NuGet 包

| 层级 | 用途 | 命名 |
|------|------|------|
| **解决方案内项目** | 维护、CI、测试引用 | 长名、带 `.SourceGenerators` / `.Package` 等后缀 |
| **NuGet 包 ID** | 应用 `PackageReference` | **仅两个**：`Observables.<Feature>.R3`、`Observables.<Feature>.Reactive` |

文档中写 `Observables.Events.R3` 时须标明指 **NuGet 包** 还是 **`Observables.Events.R3.SourceGenerators` 项目**，避免歧义。

### 每个 Feature 的项目组成

| 项目 | 是否必需 | 角色 |
|------|----------|------|
| **`Observables.<Feature>`** | 按需 | **域运行时**。纯生成、无运行时的域（如 Events）可不建。 |
| **`Observables.<Feature>.Reactive`** | 按需 | **System.Reactive 桥接运行时**（如 `IObservable` 适配器）。桥接类型放在此项目。 |
| **`Observables.<Feature>.SourceGenerators.Shared`** | 双生成器时 | 本域共享生成器逻辑（`.projitems`），由 R3 与 Reactive 两路生成器 Import。 |
| **`Observables.<Feature>.R3.SourceGenerators`** | 是 | R3 源生成器（`IsRoslynComponent`）。 |
| **`Observables.<Feature>.Reactive.SourceGenerators`** | 是 | System.Reactive 源生成器。 |
| **`Observables.<Feature>.Package`** | 发布时 | **Traversal 根** + 两个可 pack 子项目（`Observables.<Feature>.R3.csproj` 等），产出 **`Observables.<Feature>.R3`** 与 **`Observables.<Feature>.Reactive`**。Events、RestAPI 已实现；其余域待补。 |

可选扩展（**不**算第三个消费者主包）：如 `Observables.RestAPI.HttpClientFactory`，依赖域运行时，不捆绑生成器。

### 全库共享项目

| 项目 | 角色 |
|------|------|
| **`Observables.Core`** | 全库通用**运行时**（≥2 个 Feature 复用的 Attribute、枚举、接口等）。不引用 Roslyn。 |
| **`Observables.SourceGenerators.Shared`** | 全库通用**生成器**基础设施（`BackendTokens`、`GeneratedSourceHeader`、符号扩展、跨域可复用诊断如 Events `OBS2xxx`）。不引用 R3 / System.Reactive。 |
| **`Observables.Analyzers`** | 独立分析器（非生成器）：全库诊断 `OBS0001`（R3/Reactive 包冲突）、各域空代理接口 `OBS4007`/`OBS5007`/`OBS6007`/`OBS7007` 等。随 `.Package` 以 analyzer 形式分发。 |
| **`Observables.CodeFixes`** | 对应分析器/生成器诊断的 `CodeFixProvider` 与补全提供器。随 `.Package` 以 analyzer 形式分发。 |

### 反应式后端规则

| NuGet 包（目标） | 运行时 | 生成器项目 |
|------------------|--------|------------|
| **`Observables.<Feature>.R3`** | R3 | `*.R3.SourceGenerators` |
| **`Observables.<Feature>.Reactive`** | System.Reactive + 本域 `.Reactive`（若有） | `*.Reactive.SourceGenerators` |

- R3 包 **不** 引用 System.Reactive；Reactive 包 **不** 引用 R3。
- 生成器仅编译期；发布后消费者通过 **`.Package` 元包** 获得「运行时 + 对应分析器」。开发阶段用 `ProjectReference` + `OutputItemType="Analyzer"`。

### 运行时类型放在哪

```
≥2 个 Feature 复用     →  Observables.Core
仅单域使用             →  Observables.<Feature>（按需创建）
IObservable 等桥接     →  Observables.<Feature>.Reactive（按需；与 Reactive 包一起发布）
```

### `.Package` 项目（每 Feature 一个）

- **一个** `Observables.<Feature>.Package` 负责该域 **两个** NuGet 包（`PackageId` = `.R3` 与 `.Reactive`）。
- 每个包应包含：对应运行时、对应分析器 DLL、`buildTransitive` props/targets（若需要）。
- 参考 Skymly 内 `MvvmAIO.Markup.Pack`；**不要**拆成两个 `.Package` 项目（除非日后明确变更）。

### 消费者引用示例

**R3（目标 NuGet）：**

```xml
<PackageReference Include="Observables.RestAPI.R3" Version="…" />
```

**System.Reactive（目标 NuGet）：**

```xml
<PackageReference Include="Observables.RestAPI.Reactive" Version="…" />
```

开发与测试阶段：

```xml
<ProjectReference Include="..\Observables.RestAPI.R3.SourceGenerators\Observables.RestAPI.R3.SourceGenerators.csproj"
                  OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
```

### 新增 Feature 检查清单

1. 是否需要 `Observables.<Feature>` 运行时？
2. 是否需要 `Observables.<Feature>.Reactive` 桥接？
3. 建立 `*.SourceGenerators.Shared`（若两路生成器共享逻辑）
4. 建立 `*.R3.SourceGenerators` 与 `*.Reactive.SourceGenerators`（**生成器项目必须带 `.SourceGenerators` 后缀**，`.R3`/`.Reactive` 仅用于 NuGet 包 ID）
5. 建立 `*.Package`，产出 `.R3` / `.Reactive` 两个包；在 [`eng/Observables.BuildManifest.json`](eng/Observables.BuildManifest.json) 登记 `packProject` + `packageId`
6. 诊断 ID 按域分段（见「诊断治理」段分配表，如 Events `OBS2xxx`、RestAPI `OBS3xxx`、Grpc 预留 `OBS7xxx`），并在 `AnalyzerReleases.Unshipped.md` 登记
7. 补齐测试矩阵（生成器测试 + E2E + `eng/nuget-smoke` 消费者），并加入 `eng/Observables.BuildManifest.json` 的 `testProjects` / `smokeConsumers`
8. 同步文档（主仓 README、Observables.Docs、Observables.Samples）

---

## 仓库结构

```
Observables/
├── Observables.SourceGenerators.props                # 仓库根 MSBuild
├── Observables.SourceGenerators.R3.props
├── Observables.Shared/
│   ├── Observables.Core/
│   └── Observables.SourceGenerators.Shared/
├── Observables.Events/                               # 域文件夹 = Observables.<Feature>
│   ├── Observables.Events/Observables.Events.csproj  # 运行时 + targets/（同名子夹，避免 SDK  glob 同级项目）
│   ├── Observables.Events.Package/
│   ├── Observables.Events.SourceGenerators.Shared/   # shproj（双路生成器共享，#if EVENTS_R3 切换）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skymly/Observables](https://github.com/Skymly/Observables) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
