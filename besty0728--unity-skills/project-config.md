---
trigger: always_on
description: > **本文件面向"开发这个项目的 AI"**，非"调用该项目 REST API 的 AI"。
---

# UnitySkills — 项目开发者速览

> **本文件面向"开发这个项目的 AI"**，非"调用该项目 REST API 的 AI"。
> 后者请读 `SkillsForUnity/unity-skills~/SKILL.md`。

通过 REST API 让 AI 直接控制 Unity 编辑器。740 个 REST Skills + 24 个 Advisory 模块。

| 项目 | 值 |
|------|----|
| 版本 | 2.3.0 |
| 技术栈 | C# (Unity Editor Plugin) + Python (Client) |
| Unity | 2022.3+（已验证 Unity 6 / 6000.x） |
| 协议 | MIT |
| 包名 | com.besty.unity-skills |

---

## 架构

```
AI Agent ──HTTP──▶ unity_skills.py ──POST localhost:8090-8100──▶ Unity Editor Plugin
                                                                    │
                                              SkillsHttpServer (Producer-Consumer)
                                                        │
                                              SkillRouter (反射发现 [UnitySkill])
                                                        │
                                              52 个 *Skills.cs (740 Skills)
                                                        │
                                         WorkflowManager (持久化撤销/回滚)
                                         RegistryService (多实例发现)
```

**线程模型（硬约束）**：HTTP 线程仅入队，Unity 主线程通过 `EditorApplication.update` 消费。**零 Unity API 跨线程调用**——任何在 HTTP 线程上直接触发 `UnityEngine.*` / `UnityEditor.*` 的改动都属违规。

---

## 项目结构

```
Unity-Skills/
├── SkillsForUnity/                       # UPM Package (com.besty.unity-skills)
│   ├── package.json
│   ├── Editor/
│   │   ├── Skills/                       # Skill 业务层 + Server 内核
│   │   │   ├── SkillsHttpServer.cs       # HTTP 服务器 (Producer-Consumer)
│   │   │   ├── SkillRouter.cs            # 反射路由 + 参数绑定
│   │   │   ├── SkillPlanningService.cs   # SkillRouter 内部预演引擎 (/plan /dryRun + 参数语义校验，非 skill，无 [UnitySkill])
│   │   │   ├── UnitySkillAttribute.cs    # [UnitySkill] 特性 (元数据契约)
│   │   │   ├── SkillErrorResponse.cs     # 统一错误响应构建器
│   │   │   ├── SkillErrorCode.cs         # 错误码枚举
│   │   │   ├── SkillsLogger.cs           # 日志 + 版本常量唯一源 (Version)
│   │   │   ├── SkillsModeManager.cs      # 操作模式 (Approval/Auto/Bypass)
│   │   │   ├── SkillsAuditLog.cs         # 审计日志 (JSONL)
│   │   │   ├── ConfirmationTokenService.cs  # 高危确认 token
│   │   │   ├── WorkflowManager.cs        # 持久化工作流
│   │   │   ├── RegistryService.cs        # 全局多实例注册表
│   │   │   ├── GameObjectFinder.cs       # 统一查找器 (name/instanceId/path)
│   │   │   ├── BatchExecutor.cs          # 批量操作框架
│   │   │   ├── SkillInstaller.cs         # AI 工具一键安装
│   │   │   ├── UnityCliService.cs        # Unity CLI 检测 + 项目绑定 (Library/UnitySkills/cli_config.json)
│   │   │   └── *Skills.cs × 52           # 功能模块 (共 740 Skills)
│   │   └── UI/                           # Editor UI (USS + UXML + EditorWindow)
│   │       ├── UnitySkillsWindow.{cs,uxml,uss}    # 主窗口
│   │       ├── AuditLogWindow.{uxml,uss}          # 审计窗口
│   │       ├── AllowlistPickerWindow.{uxml,uss}   # 白名单挑选
│   │       ├── UnityCliWindow.{cs,uxml,uss}       # Unity CLI 配置二级面板
│   │       └── Tabs/*.uxml                        # 标签页与抽屉
│   └── unity-skills~/                    # AI Skill 模板（波浪线隐藏，随包分发）
│       ├── SKILL.md                      # 调用方文档（"用"项目）
│       ├── scripts/unity_skills.py
│       ├── skills/                       # 72 个模块文档 (48 REST + 24 advisory)
│       └── references/
├── .claude/commands/                     # 自定义命令
├── docs/SETUP_GUIDE.md
├── CHANGELOG.md
└── agent.md                              # 本文件（"写"项目）
```

---

## 编写约束

### 1. Editor UI：USS + UXML（硬约束）

- **禁止 IMGUI**：不允许 `OnGUI` / `EditorGUILayout` / `GUILayout` / `Editor.OnInspectorGUI` 等任何 IMGUI API。已有窗口全部基于 UI Toolkit，新增 UI 必须沿用。
- **文件三件套**统一放 `Editor/UI/`：`XxxWindow.cs` + `XxxWindow.uxml` + `XxxWindow.uss`。
- **加载范式**（参考 `UnitySkillsWindow.cs`）：
  1. 用 `Packages/com.besty.unity-skills/Editor/UI/Xxx.{uxml,uss}` 绝对包路径常量
  2. `CreateGUI()` 内先 `styleSheets.Add(uss)` 再 `uxml.CloneTree(rootVisualElement)`
  3. 节点用 `rootVisualElement.Q<T>("element-name")` 获取
  4. 周期刷新走 `schedule.Execute(...).Every(ms)`，**禁止** `EditorApplication.update` 轮询 UI
- **复杂窗口拆 Controller**：每个 UXML 子树由一个 `XxxController(VisualElement root, EditorWindow owner)` 接管，主窗口仅做组装与生命周期；参考 `TopbarController` / `SkillsTabController` / `SettingsDrawerController`。
- **Window 菜单单入口（用户拍板）**：`[MenuItem("Window/UnitySkills")]` 是叶子项且全项目唯一——Unity 里叶子与 `Window/UnitySkills/...` 子菜单不能并存（叶子会被吞）。**禁止**在该前缀下新增任何 `[MenuItem]`；二级面板（审计日志）只走面板内按钮 + 快捷键。
- **面板快捷键**：命令统一注册在 `Editor/UI/ShortcutActions.cs`（Unity `[Shortcut]` 特性，出厂默认不绑定）。新增可绑定面板 = 加一个 `[Shortcut]` 静态方法 + `Commands` 清单追加一条，设置抽屉 Shortcuts 节自动枚举；用户主动设置的绑定由 ShortcutManager profile 持久化，不写 EditorPrefs。
- **样式只走 USS class**：不要在 C# 里手写 `style.color = ...`，改类名（`AddToClassList` / `RemoveFromClassList`）。

### 2. Skill 编写规范

```csharp
[UnitySkill("skill_name", "描述（首句给 AI 看，建议带参数说明）",
    Category   = SkillCategory.GameObject,
    Operation  = SkillOperation.Create,        // 多操作可用 | 合并
    Tags       = new[] { "primitive", "hierarchy" },
    Outputs    = new[] { "instanceId", "path" },
    TracksWorkflow   = true,                   // 写型 skill 强烈建议
    MutatesScene     = true,                   // 风险元数据按实际填
    RiskLevel        = "medium")]              // low / medium / high
public static object SkillName(string name, float x = 0f) { ... }
```

- **必填**：Name（构造器位置参）、Description、`Category`、`Operation`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Besty0728/Unity-Skills](https://github.com/Besty0728/Unity-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
