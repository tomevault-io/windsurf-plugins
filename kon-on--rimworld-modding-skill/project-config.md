---
trigger: always_on
description: |
---


# RimWorld Mod 制作指南

## 🧠 核心决策：何时调 MCP，何时直接用模板

**三层决策——每次接收 mod 制作请求时，按以下优先级判断：**

```
用户请求
  │
  ├─ ① 有模板？ ─── 武器/服装/建筑/资源/配方/Harmony/ThingComp
  │     └─ ✅ 直接用模板生成，不调 MCP（模板已验证过原版结构）
  │
  ├─ ② 报错/调试？ ─── 红字/白窗/崩溃/NullReferenceException
  │     └─ 🔍 调 MCP 查错误原因、搜索类似问题（MCP 不可用则 grep）
  │
  └─ ③ 无模板的新类型？ ─── 植物/生物/派系/事件/地形/Hediff/研究...
        └─ 📡 调 MCP 查原版 Def 结构 → 模仿写出 Def → 存储为新模板
```

### 模板 ↔ Def 类型对照表

| 用户需求 | 对应模板 | MCP？ |
|----------|---------|:----:|
| 近战武器 | `templates/weapon-melee.xml` | ❌ 免 |
| 远程武器 | `templates/weapon-ranged.xml` | ❌ 免 |
| 服装/护甲/头饰 | `templates/apparel.xml` | ❌ 免 |
| 原材料/建筑材料 | `templates/resource-stuff.xml` | ❌ 免 |
| 建筑/工作台 | `templates/building.xml` | ❌ 免 |
| 制作配方 | `templates/recipe.xml` | ❌ 免 |
| Harmony 补丁 | `templates/harmony-patch.cs` | ❌ 免 |
| C# ThingComp | `templates/thingcomp.cs` | ❌ 免 |
| 修改原版（XML） | `references/04-xml-patching.md` | ❌ 免 |
| **植物 / 生物 / 派系 / 事件** | **无模板** | 📡 MCP |
| **地形 / Hediff / 研究 / 工作** | **无模板** | 📡 MCP |
| 任何**错误排查** | `references/08-debugging.md` | 🔍 MCP |

---

## ⚙️ RimSage MCP 配置

> 如果还没有配置，参考 `.mcp.json.example` 创建项目级配置，或运行平台对应的 MCP 添加命令。
> 首次索引需 30-60 分钟。详见 `references/11-platform-adaptation.md`。

**MCP 不可用时**：降级为本地 `grep` 搜索 `RimWorld/Data/Core/Defs/`。

### MCP 三件套（仅在上表标明 📡 或 🔍 时使用）

| 场景 | 用这个工具 | 示例 |
|------|-----------|------|
| 无模板 → 查原版 Def 结构 | `get_def_details` | `get_def_details("Plant_TreeOak")` |
| 无模板 → 搜索实现模式 | `search_rimworld_source` | `"plant def wild cluster"` |
| 报错 → 查错误原因 | `search_rimworld_source` | `"NullReferenceException food"` |

### 新类型模板创建流程（场景 ③）

```
用户要求无模板类型
  → ① MCP 查原版类似 Def（get_def_details + search_rimworld_source）
  → ② 写出完整可用的 Def（带中文注释，遵循下方"模板格式规范"）
  → ③ Write 模板到文件系统（绝对路径，不能用对话中输出代替）
  → ④ 更新 SKILL.md 两处列表（见下方"存储与更新清单"）
  → ✅ 下次同类型请求直接免 MCP
```

**📋 存储与更新清单（每次创建新模板后必须逐项完成）：**

| # | 操作 | 工具 | 目标位置 |
|---|------|------|---------|
| 1 | 写入模板文件 | `Write` | `templates/<新类型>.xml`（或用 `.cs` 后缀） |
| 2 | 添加到"模板 ↔ Def 类型对照表" | `Edit` | SKILL.md 顶部表格——新增一行，标记 `❌ 免` |
| 3 | 添加到"子文件索引 → 代码模板" | `Edit` | SKILL.md 底部 `templates/` 列表——新增条目 |

**📐 模板格式规范（Write 模板文件时必须遵守）：**

1. `<?xml version="1.0" encoding="utf-8"?>` 开头（XML 模板）
2. 注释块（`<!-- ... -->`）包含：
   - 模板类型名称 + 验证状态
   - 原版参考路径（哪个原版 XML 文件）
   - ParentName 继承链（完整层级）
   - 关键设计决策说明
3. 所有用户自定义值使用 `<YourXxx>` 占位符（如 `<YourPrefix>_Thing_<YourName>`）
4. 关键字段附带中文行内注释（含可选值列表/合法范围）
5. 文件末尾保留一个空行

**💡 模板示例参考**：打开任意现有模板（如 `templates/weapon-melee.xml`）查看注释头和占位符风格，新模板应与之保持一致。

---

## 🧠 学习系统

每次加载此 Skill 时，必须先读取 `learnings/errors.txt` 中的历史错误记录，
并在后续工作中避免重复犯错。

排查完错误并修复后，必须将关键教训总结为一句话追加到 `learnings/errors.txt`。
格式：`YYYY-MM-DD | <类别> | <一句话总结>`

| 类别 | 适用场景 |
|------|---------|
| XML | ThingDef/RecipeDef/Def 属性写错、枚举值错误、ParentName 错误 |
| C# | 编译错误、空引用、类型错误、DLL 加载问题 |
| Harmony | Patch ID 冲突、补丁未生效、Prefix/Postfix 签名错误 |
| Path | 文件路径错误、资源缺失、加载顺序问题 |
| Other | 上述类别之外的错误 |

此文件仅在 rimworld-modding skill 上下文中读取，其他场景不读取。

---

## 核心工作流：测试版先行（Test → Verify → Formalize）

**所有 mod 生成后，默认先生成测试版本。开发者确认无 bug 后，再正规化并可选发布。**

```
用户需求 → 生成测试版 Mod → 开发者进游戏测试
                              ├─ 有 bug → 修复 → 重新测试
                              └─ 确认无误 → 正规化 → 可选：上传 Steam
```

> **测试版 vs 正规版**：测试版快速迭代验证；正规版清理命名、补全 About.xml、准备发布。

## 快速导航

根据你的需求，我会自动加载对应的参考文档和模板。

### 我想...
| 需求 | 加载内容 |
|------|---------|
| 🆕 **新建一个 mod 项目** | `workflows/new-mod.md` + `references/02-project-structure.md` |
| ⚔️ **添加武器/物品/建筑/服装/植物** | `references/03-xml-defs.md` + 对应 `templates/*.xml` |
| 🔧 **修改原版机制/打补丁** | `references/04-xml-patching.md`（XML）/ `references/06-harmony.md`（C#） |
| 💻 **编写 C# 代码/DLL** | `references/05-csharp-basics.md` |
| 🎵 **用 Harmony 拦截方法** | `references/06-harmony.md` + `templates/harmony-patch.cs` |
| 🖼️ **添加纹理/音效资源** | `references/07-assets.md` |
| 🐛 **排查报错/崩溃/红字** | `references/08-debugging.md` |
| 📖 **查看历史错误记录** | `learnings/errors.txt`（自动读取） |
| ✅ **测试通过，正规化 mod** | `workflows/formalize-mod.md` |
| 📦 **批量处理多个需求** | `workflows/batch-process.md` + `templates/requirements-template.md` |
| 📦 **发布到 Steam Workshop** | `references/09-workshop.md` |
| 📖 **查询 API/类/方法** | `references/10-api-reference.md`（建议先接入 RimSage MCP） |
| 🔄 **适配其他 AI 平台** | `references/11-platform-adaptation.md`（Copilot/Codex/Gemini 等工具映射） |

## 核心原则

### 1. 命名规范
- **前缀**：所有 defName 和 C# 类名使用唯一前缀（**由用户自行选择**，如 `RCS_`、`AWE_` 等），避免 mod 冲突
- **namespace**：`你的前缀.Mod名`（由用户根据前缀确定）
- **packageId**：`你的用户名.mod名`（全小写，无空格，**用户自行填写**，如 `steamid.modname`）
- **作者名**：所有 `<author>`、作者字段**必须由用户自行填写**，AI 不预设作者信息

### 2. 版本注意
- **目标版本**：RimWorld 1.6
- **Unity 版本**：2022.3.35
- **.NET Framework**：4.7.2+
- 每个 reference 文件顶部标注适用版本和最后更新日期

### 3. 安全实践
- 始终在 `[StaticConstructorOnStartup]` 中初始化 Harmony
- Harmony patch 方法使用唯一的 patch ID
- ExposeData 中正确保存/加载自定义数据
- 避免在构造函数中做重型操作
- 优先使用 PatchOperations 而非直接修改原版文件

### 4. ⚖️ 法律边界——什么能做，什么不能

**Ludeon Studios 积极支持 mod 制作**——官方 Wiki 有完整 modding 教程，Steam Workshop 是官方集成渠道。但以下底线不可越过：

#### ✅ 完全合法——放心做

| 行为 | 依据 |
|------|------|
| 编写原创 XML Def | RimWorld 公开数据接口 |
| 用原版 ParentName 继承 | 等同于调用公开 API |
| 模仿原版 Def 结构 | 等同于参考 API 文档 |
| 用 dnSpy / MCP 查原版代码 | mod 社区标准做法（官方 Wiki C# 教程第一页就教你用 dnSpy） |
| 发布原创 mod 到 Steam Workshop | 官方支持的发布渠道 |

#### ❌ 绝对禁止

| 行为 | 原因 |
|------|------|
| **复制原版 C# 源码到你的 mod** | 侵犯 Ludeon 版权 |
| **打包原版 DLL（Assembly-CSharp.dll 等）到你的 mod** | 侵犯版权 |
| **复制其他 mod 的代码/资产/贴图** | 侵犯原作者权利 |
| **使用第三方 IP（宝可梦、星战、漫威等）的角色/名称/资产** | 商标/版权侵权 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kon-on/rimworld-modding-skill](https://github.com/Kon-on/rimworld-modding-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
