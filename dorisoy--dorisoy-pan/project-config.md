---
trigger: always_on
description: 本说明适用于新生命团队（NewLife）及其全部开源/衍生项目，规范 Copilot 及类似智能助手在 C#/.NET 项目中的协作行为。
---

# NewLife Copilot 协作指令

本说明适用于新生命团队（NewLife）及其全部开源/衍生项目，规范 Copilot 及类似智能助手在 C#/.NET 项目中的协作行为。

> 目标：把"每次请求必须携带的通用规则"控制在可接受体积；组件/业务专项流程放在 `.github/instructions/`，按需读取。

---

## 1. 核心原则

| 原则 | 说明 |
|------|------|
| **提效** | 减少机械样板，聚焦业务/核心算法 |
| **一致** | 风格、结构、命名、API 行为稳定 |
| **可控** | 限制改动影响面，可审计，兼容友好 |
| **可靠** | 先检索再生成，不虚构，不破坏现有合约 |
| **主动** | 发现问题主动修复，不回避合理优化 |

---

## 2. 适用范围

- 含 NewLife 组件或衍生的全部 C#/.NET 仓库
- 不含纯前端/非 .NET/市场文案
- 存在本文件 → 必须遵循

---

## 3. 组件专用指令索引（按需加载）

以下专用指令**仅在相关任务时**才需要读取，避免每次请求都携带大段流程/示例。

### 3.1 XCode / Cube（数据库 & Web 快速开发）

当任务涉及以下任一信号时，请**先搜索并检查当前仓库** `.github/instructions/xcode.instructions.md` **是否存在**，若存在则读取并遵循：

- 需求包含：XCode/Cube/魔方/实体生成/模型 XML/数据类库/数据库 CRUD/Controller 生成/`xcodetool`/`xcode` 命令
- 解决方案/项目中出现：`NewLife.XCode` 包引用
- 存在：`Model.xml`、`*.xcode.xml`、`*.Data.csproj`（或项目名以 `.Data` 结尾）
- 代码出现命名空间/类型：`XCode.*`、`Entity`（XCode 实体基类）、XCode 相关特性/接口
- **用户提到修改任意 `.xml` 文件**（如 `member.xml`、`area.xml` 等配置文件），应**主动搜索** `xcode.instructions.md` 判断是否需要引入

**主动检测策略**：当用户提及 XML 文件修改时，即使未明确提到 XCode 关键字，也应先用 `file_search` 搜索 `xcode.instructions.md`，若存在则读取，以确定该 XML 文件是否属于 XCode/Cube 体系。

未满足以上条件时，**不要**引入 XCode/Cube 初始化流程，避免干扰其它仓库的常规开发。

---

## 4. 工作流

```
需求分类 → 检索 → 评估 → 设计 → 实施 → 验证 → 说明
```

1. **需求分类**：功能/修复/性能/重构/文档
2. **检索**：相关类型、目录、方法、已有扩展/工具（**优先复用**）
3. **评估**：是否公共 API？是否性能热点？**是否存在潜在问题？**
4. **设计**：列出改动点 + 兼容/降级策略
5. **实施**：
   - 完成用户请求的核心任务
   - **顺带修复**发现的明显缺陷（资源泄漏、空引用、逻辑错误）
   - **顺带优化**可简化的重复代码
   - 保留原注释与结构，除非注释本身有误
6. **验证**：
   - 代码变更：必须编译通过；运行相关单元测试（未找到需说明）
   - 仅文档变更（未修改任何代码文件）：可跳过编译与单元测试
7. **说明**：变更摘要/影响范围/风险点

### 4.1 主动优化原则

当用户请求分析或优化代码时，**应主动**：

| 类型 | 行动 |
|------|------|
| **架构梳理** | 梳理代码架构并进行重构，让代码结构更清晰易懂 |
| **语法现代化** | 使用最新的 C# 语法来简化代码，提升可读性 |
| **缺陷修复** | 资源泄漏、空引用风险、并发问题、逻辑错误 → 直接修复，让代码更健壮 |
| **性能优化** | 无用分配、重复计算、可池化资源 → 通过缓存减少耗时的重复计算 |
| **代码简化** | 重复代码提取、冗余判断合并、现代语法替换 → 在不影响可读性前提下简化 |
| **注释完善** | 补充类、接口、属性、方法头部的注释，以及方法内部重要代码的注释 |
| **架构参考** | 参考网络上同类功能的优秀架构，给出架构调整建议 |

**架构调整策略**：
- **改动较小**：直接调整，完成后说明变更内容
- **改动较大**：先列出调整方案，询问用户意见，待确认后再修改

**不应过度保守**：
- ❌ 仅添加注释而忽略明显的代码问题
- ❌ 发现资源泄漏却不修复
- ❌ 看到重复代码却不提取
- ❌ 用户要求优化时只做表面工作

**保持谨慎的场景**：
- 公共 API 签名变更 → 需说明兼容性影响
- 性能关键路径 → 需有依据或说明推理
- 大范围重构 → 需先与用户确认范围

### 4.2 防御性注释规则

在旧有代码中，经常可以看到**被注释掉的代码**，这些注释代码前面通常带有说明文字。

**这些是防御性注释**：
- 记录了过去曾经踩过的坑
- 目的是告诉后来人不要按照注释代码去写，否则会有问题
- **禁止删除此类防御性注释**，用于警示后人

**识别特征**：
```csharp
// 曾经尝试过 xxx 方案，但会导致 yyy 问题
// var result = DoSomethingWrong();

// 不要使用 xxx，否则会造成 yyy
// await client.SendAsync(data);

// 这里不能用 xxx，因为 yyy
// stream.Flush();
```

**处理原则**：
- ✅ 保留这类带说明的注释代码
- ✅ 可以补充更详细的说明，解释为什么不能这样做
- ❌ 不要删除这类防御性注释
- ❌ 不要尝试"恢复"这些被注释的代码

---

## 5. 编码规范

### 5.1 基础规范

| 项目 | 规范 |
|------|------|
| 语言版本 | `<LangVersion>latest</LangVersion>`，所有目标框架均使用最新 C# 语法 |
| 命名空间 | file-scoped namespace |
| 类型名 | **必须**使用 .NET 正式名 `String`/`Int32`/`Boolean` 等，避免 `string`/`int`/`bool` |
| 兼容性 | 代码需兼容 .NET 4.5+；**禁止**使用 `ArgumentNullException.ThrowIfNull`，改用 `if (value == null) throw new ArgumentNullException(nameof(value));` |
| 单文件 | 每文件一个主要公共类型；较大平台差异使用 `partial` |

### 5.2 命名规范

| 成员类型 | 命名规则 | 示例 |
|---------|---------|------|
| 类型/公共成员 | PascalCase | `UserService`、`GetName()` |
| 参数/局部变量 | camelCase | `userName`、`count` |
| 私有字段（实例/静态） | `_camelCase` | `_cache`、`_instance` |
| 属性/方法（实例/静态） | PascalCase | `Name`、`Default`、`Create()` |
| 扩展方法类 | `xxxHelper` 或 `xxxExtensions` | `StringHelper`、`CollectionExtensions` |

### 5.3 代码风格

```csharp
// ✅ 单行 if：单语句且整行不过长时同行
if (value == null) return;
if (key == null) throw new ArgumentNullException(nameof(key));

// ✅ 单行 if：语句较长时另起一行
if (value == null)
    throw new ArgumentNullException(nameof(value), "Value cannot be null");

// ✅ 多分支单语句：不加花括号
if (count > 0)
    DoSomething();
else
    DoOther();

// ✅ 循环必须保留花括号（即使单语句）
foreach (var item in list)
{
    Process(item);
}
```

### 5.4 Region 组织结构

较长的类使用 `#region` 分段组织，顺序为：`属性` → `静态`（如有）→ `构造` → `方法` → `辅助`（如有）→ `日志`。

**日志 Region 规则**：
- 类代码中如果带有 `ILog Log { get; set; }` 和 `WriteLog` 方法
- **必须放在类代码的最后**
- **必须用名为"日志"的 region 包裹**
- 不要放在"辅助" region 中，应单独作为"日志" region

### 5.5 现代 C# 语法

优先使用最新语法（switch 表达式、模式匹配、目标类型 `new`、record 等），即使目标框架是 net45。

### 5.6 集合表达式

优先使用集合表达式 `[]` 初始化集合：`List<String> Tags { get; set; } = [];`

### 5.7 Null 条件运算符

优先使用 `?.` / `??` 简化空值检查：`span?.AppendTag("test");` `var name = user?.Profile?.Name ?? "";`

---

## 6. 多目标框架

NewLife 支持 `net45` 到 `net10`，常用条件符号：`NETFRAMEWORK`、`NETSTANDARD2_0`、`NETCOREAPP`、`NET5_0_OR_GREATER`、`NET6_0_OR_GREATER`、`NET8_0_OR_GREATER`。

新增 API 时需评估各框架兼容性，必要时提供降级实现。

---

## 7. 文档注释

| 规则 | 说明 |
|------|------|
| `<summary>` | **必须同一行闭合**，简短描述方法用途 |
| `<param>` | **必须为每个参数添加**，无论方法可见性如何 |
| `<returns>` | 有返回值时必须添加 |
| `<remarks>` | 复杂方法可增加详细说明（可多行） |
| 覆盖范围 | `public`/`protected` 成员必须注释；`private`/`internal` 建议添加 |
| `[Obsolete]` | 必须包含迁移建议 |

**正确示例**：`/// <summary>获取名称</summary>` `/// <param name="id">编号</param>`

**禁止**：`<summary>` 拆成多行；缺少 `<param>`；有参数但无 param 标签。

---

## 8. 异步与性能

| 规范 | 说明 |
|------|------|
| 方法命名 | 异步方法后缀 `Async` |
| ConfigureAwait | 库内部默认 `ConfigureAwait(false)` |
| 高频路径 | 优先对象池/`ArrayPool<T>`/`Span`，避免多余分配 |
| 反射/Linq | 仅用于非热点路径；热点使用手写循环/缓存 |
| 池化资源 | 明确获取/归还；异常分支不遗失归还 |

**内置工具优先**：`Pool.StringBuilder`、`Runtime.TickCount64`、`ToInt()`/`ToBoolean()` 等扩展方法。

---

## 9. 日志与追踪


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorisoy/Dorisoy.Pan](https://github.com/dorisoy/Dorisoy.Pan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
