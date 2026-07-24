---
trigger: always_on
description: 适用于 NewLife 系列全部 C#/.NET 仓库，本文件可随仓库/指令目录一起拷贝到其他项目直接复用。存在本文件则必须遵循。**简体中文回复。**
---

# NewLife Copilot 协作指令

适用于 NewLife 系列全部 C#/.NET 仓库，本文件可随仓库/指令目录一起拷贝到其他项目直接复用。存在本文件则必须遵循。**简体中文回复。**
通用 C# 最佳实践（设计模式、SOLID、健壮性等）AI 已知，此处不赘述，**仅列出组织专属规则与反常规约定**。

---

## 1. 专用指令（前置检查，必须执行）

**开始任何任务前，必须先将用户请求与下表触发信号逐行匹配。命中则立即用 `get_file` 读取 `.github/instructions/{指令文件}`，读取成功后遵循其中全部规则。未命中任何行才跳过。**

| 触发信号（用户请求含以下任意关键词即命中） | 指令文件 |
|---------|---------|
| XCode/实体生成/Model.xml/数据库 CRUD/`NewLife.XCode` 引用/`*.xcode.xml`/项目名含 `.Data`/`XCode.*` 命名空间/用户提及修改任意 `.xml` 文件 | `xcode.instructions.md` |
| Cube/魔方/Web开发/`NewLife.Cube` 引用/`NewLife.Cube.*` 命名空间 | `cube.instructions.md` |
| 性能测试/基准测试/压力测试/压测/BenchmarkDotNet/Benchmark/benchmark/吞吐量评估/性能分析/性能对比/性能报告/速度对比/速度测试/内存分配/perf/性能优化测试/做性能/跑分/测试报告 | `benchmark.instructions.md` |
| NetServer/NetSession/网络服务器/网络客户端/Socket服务/TCP服务/UDP服务/`NewLife.Net` 引用/`NewLife.Net.*` 命名空间/ISocketClient/ISocketRemote/CreateRemote/StandardCodec/LengthFieldCodec/管道编解码/网络编程/Echo服务/网络会话/长连接/粘包拆包 | `net.instructions.md` |
| 新建系统/新建项目/新增模块/需求整理/需求文档/需求分析/架构设计/技术方案/功能清单/功能拆分/任务分解/迭代开发/迭代计划/验收/PRD/用户故事/做一个系统/做一个平台/开发流程/全部搞完/批量开发/自治模式/一次性做完/继续处理/接着做 | `development.instructions.md` |
| 缓存/ICache/MemoryCache/Redis缓存/ICacheProvider/缓存设计/`NewLife.Caching` 命名空间 | `caching.instructions.md` |
| 序列化/JSON/Binary/JsonHelper/序列化设计/SpanSerializer/CSV导出/`NewLife.Serialization` 命名空间 | `serialization.instructions.md` |
| 加密/安全/Hash/MD5/SHA/AES/SM4/RSA/JWT/SecurityHelper/TokenProvider/`NewLife.Security` 命名空间 | `security.instructions.md` |
| 远程调用/ApiHttpClient/ApiClient/ApiServer/负载均衡/LoadBalancer/RPC/HTTP客户端/`NewLife.Remoting` 命名空间 | `remoting.instructions.md` |
| 配置/Config/IConfigProvider/HttpConfigProvider/CommandParser/配置中心/`NewLife.Configuration` 命名空间 | `configuration.instructions.md` |

**自动匹配指令**（无需触发，按 `applyTo` 路径自动生效）：`caching`、`serialization`、`security`、`remoting`、`configuration` 这 5 个指令文件同时配置了 `applyTo` 模式，编辑对应目录下的文件时 VS Code 会自动加载。

---

## 2. 核心原则

检索优先、风格一致、兼容友好、**主动优化**。
发现明显缺陷（资源泄漏、空引用、逻辑错误）时主动修复；优化请求时深入分析，不做表面工作。
改动较小直接做并说明；改动较大（涉及公共 API 或大范围重构）先列方案询问确认。

---

## 2.1 项目双版本架构（极重要）

本仓库包含两个代际不同的项目，**定位与维护策略截然不同**：

| 项目 | 代际 | 定位 | 维护策略 |
|------|------|------|---------|
| **`NewLife.Cube`** | **第三代（主维护）** | API 前后端分离版本，前端皮肤独立 | **新增功能优先在此实现**，长期持续维护 |
| **`NewLife.CubeNC`** | 第二代（历史遗留） | MVC 服务端渲染版本 | 逐步减少维护，通过文件链接复用第三代代码 |

**文件链接方向（单向）：**
- `NewLife.CubeNC.csproj` → 链接 `NewLife.Cube` 的 `Services/`、`Jobs/`、`Extensions/`、`Entity/`、`Common/` 等文件

**开发原则：**
1. **新功能必须写在 `NewLife.Cube`**（第三代），不在 CubeNC 独立实现
2. 需要让 MVC 版（CubeNC）同步获得新功能时，在 `NewLife.CubeNC.csproj` 的 `<ItemGroup>` 中添加 `<Compile Include>` 指向 `NewLife.Cube` 的文件链接
3. CubeNC 有独立的 `Controllers/CubeController.cs`、`Controllers/SsoController.cs` 等，若需同步逻辑，应优先考虑链接而非复制
4. 两个项目均有独立的 `Setting.cs`；**新配置项先加在 `NewLife.Cube/Setting.cs`**，再视需要在 CubeNC 侧通过文件链接或手动同步

---

## 3. 兼容性约束（极重要）

- **语言版本**：当前为 **C# 14**（`<LangVersion>latest</LangVersion>`），最大化使用最新语法糖（switch 表达式、集合表达式 `[]`、`?.`/`??`/`??=`、模式匹配、目标类型 `new`、record 等）
- **框架版本**：新增 API 前，先查看当前项目 `.csproj` 的 `<TargetFrameworks>` 配置，**只需满足已声明版本的兼容性**，无需对所有历史版本降级。若包含 `net45`/`netstandard2.0` 等低版本，再提供条件编译降级实现。
- **禁止高版本专属 BCL API**（低版本项目）：❌ `ArgumentNullException.ThrowIfNull()` → ✅ `if (x == null) throw new ArgumentNullException(nameof(x));`
- **条件编译符号**：`NETFRAMEWORK`、`NETSTANDARD2_0`、`NETCOREAPP`、`NET5_0_OR_GREATER`、`NET6_0_OR_GREATER`、`NET8_0_OR_GREATER`

---

## 4. 编码规范

### 4.1 类型名（关键差异）

**必须**使用 .NET 正式名：`String`/`Int32`/`Boolean`/`Int64`/`Double`/`Object` 等。
❌ **禁止**使用 C# 别名：`string`/`int`/`bool`/`long`/`double`/`object`

### 4.2 命名

| 成员类型 | 规则 | 示例 |
|---------|------|------|
| 类型/公共成员 | PascalCase | `UserService`、`GetName()` |
| 参数/局部变量 | camelCase | `userName`、`count` |
| 私有字段 | `_camelCase` | `_cache`、`_instance` |
| 扩展方法类 | `xxxHelper` 或 `xxxExtensions` | `StringHelper`、`CollectionExtensions` |

### 4.3 代码风格

- **命名空间**：file-scoped namespace
- **单文件**：每文件一个主要公共类型；较大平台差异使用 `partial`
- **集合初始化**：优先使用集合表达式 `[]`，如 `List<String> Tags { get; set; } = [];`
- **Null 条件运算符**：优先使用 `?.`/`??` 简化空值检查；**C# 14 空条件赋值 `??=`**：变量为 null 时才赋值，可显著提升可读性

```csharp
// ✅ C#14 空条件赋值（??=）：为 null 时才赋值，替代 if (x == null) x = ...
_cache ??= new MemoryCache();
list ??= [];

// ✅ if 内只有单行代码时可不加花括号（单行 if 同行或换行均可）
if (value == null) return;
if (key == null) throw new ArgumentNullException(nameof(key));

// ✅ 语句较长时另起一行，仍不加花括号
if (value == null)
    throw new ArgumentNullException(nameof(value), "Value cannot be null");

// ✅ 多分支单语句：不加花括号
if (count > 0)
    DoSomething();
else
    DoOther();

// ✅ for/foreach/while 循环体必须保留花括号（即使单语句）
foreach (var item in list)
{
    Process(item);
}

for (var i = 0; i < count; i++)
{
    Process(i);
}

// ✅ using 优先无花括号声明；仅需生命周期（如锁）时用弃元
using var stream = File.OpenRead("file.txt");
using var _ = _lock.AcquireLock();
```

### 4.4 Region 与日志

较长类使用 `#region` 分段，顺序：`属性` → `静态` → `构造` → `方法` → `辅助` → **`日志`**。
含 `ILog Log` 和 `WriteLog` 时：**必须放类末尾**，用名为"日志"的 region 包裹，不放入"辅助"。
关键过程可使用 `Tracer?.NewSpan()` 埋点。

### 4.5 文档注释

- `<summary>` **必须同行闭合**：`/// <summary>获取名称</summary>`
- 每个参数**必须有** `<param>` 标签，无论方法可见性
- 有返回值**必须有** `<returns>`；复杂方法可增加 `<remarks>`
- `public`/`protected` 成员必须注释；`[Obsolete]` 必须包含迁移建议

### 4.6 异步与性能

- 异步方法后缀 `Async`，库内部默认 `ConfigureAwait(false)`
- 热点路径避免反射/复杂 Linq，优先手写循环/`ArrayPool<T>`/`Span`
- 池化资源明确获取/归还，异常分支不遗失归还

### 4.7 错误处理


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NewLifeX/NewLife.Cube](https://github.com/NewLifeX/NewLife.Cube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
