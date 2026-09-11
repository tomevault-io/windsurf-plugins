---
trigger: always_on
description: > 本文件为 AI 智能体（及人类开发者）提供项目导航与编码规范。
---

# AGENTS.md

> 本文件为 AI 智能体（及人类开发者）提供项目导航与编码规范。
> 遵循本文档可确保代码风格一致、主题正确、组件归属清晰。

---

## 项目概览

**daro** 是一个用 Flutter 构建的 桌面数据库管理工具。
目标是高信息密度、桌面级交互体验，支持明 / 暗双主题。

| 维度 | 说明 |
|---|---|
| 框架 | Flutter (Dart SDK ≥3.0) |
| 状态管理 | `provider` + `ChangeNotifier` (AppState) |
| 主题 | 自建 `AppPalette` / `Tokens` 语义色板 + `base_ui_flutter` 的 `DesktopTokens` / `TokenScope` |
| UI 组件库 | `base_ui_flutter`（本地 path 依赖，git submodule） |
| 字体 | `chinese_font_library` 中文排版优化 |

---

## 目录结构

```
daro/
├── lib/
│   ├── main.dart                 # 应用入口；TokenScope 包裹；主题切换
│   ├── app/
│   │   └── app_state.dart        # AppState: 主题模式、标签管理、表选择
│   ├── data/
│   │   └── db_data.dart     # 静态模拟数据（连接 / 数据库 / 表列表）
│   ├── pages/
│   │   └── main_page.dart     # 主页面布局：顶栏 + Ribbon + 三栏 + 状态栏
│   ├── theme/
│   │   └── app_theme.dart        # AppPalette 明暗色板 + Tokens.of() + toDesktopTokens() 桥接
│   └── widgets/                  # 应用级 widget（业务耦合，不可复用）
│       ├── top_menu.dart         # 顶部菜单栏(复用 base-ui-flutter MenuStrip) + 主题切换按钮
│       ├── ribbon.dart           # 工具栏
│       ├── database_tree.dart    # 左侧连接树
│       ├── object_panel.dart    # 中部对象面板（108 表网格）
│       ├── object_tabs.dart      # 对象路径标签
│       ├── view_tabs.dart        # 视图标签栏
│       ├── database_info.dart    # 右侧详情面板
│       ├── status_bar.dart       # 底部状态栏
│       ├── table_icon.dart       # 表图标（应用级，不归入 base-ui-flutter）
│       ├── table_data_page.dart  # 表数据浏览页
│       └── query_page.dart       # SQL 查询编辑页
│
├── base-ui-flutter/             # 独立 UI 组件库（git submodule）
│   └── lib/
│       ├── base_ui_flutter.dart  # barrel export（单一入口）
│       └── src/
│           ├── foundation/       # DesktopTokens、TokenScope、Control 基类
│           ├── common/            # Button、Input、Label、CheckBox、ComboBox…
│           ├── lists/             # ListBox、TreeView、DataGridView…
│           ├── containers/         # GroupBox、TabControl、SplitContainer…
│           ├── menus/             # MenuStrip、ToolStrip、StatusStrip…
│           ├── overlay/           # Popover、MessageBox、Toast…
│           ├── dialogs/           # ColorDialog、DateTimePicker…
│           ├── data/              # Chart、Pagination…
│           ├── scroll/            # ScrollBar、TrackBar
│           └── misc/              # ProgressBar、Skeleton、Spinner…
│
├── pubspec.yaml                  # base_ui_flutter 通过 path 依赖引入
└── AGENTS.md                     # 本文件
```

---

## 双层主题系统

### 第一层：AppPalette / Tokens（应用语义色板）

定义在 `lib/theme/app_theme.dart`。这是应用的**主**主题系统，包含 应用特有的语义色（`connIcon`、`schemaIcon`、`folderIcon`、`tableIcon` 等）。

```dart
// 取色方式
final t = Tokens.of(context);
Container(color: t.surface);
Text('hello', style: TextStyle(color: t.textPrimary));
```

### 第二层：DesktopTokens / TokenScope（base_ui_flutter 令牌）

`base_ui_flutter` 组件库使用自己的 `DesktopTokens` 系统。通过 `AppPalette.toDesktopTokens()` 桥接方法，将应用色板映射到 `DesktopTokens` 字段，使 `base_ui_flutter` 组件自动跟随应用主题。

```dart
// 在 main.dart 中
home: Builder(
  builder: (context) {
    final brightness = Theme.of(context).brightness;
    final palette = brightness == Brightness.dark
        ? AppTheme.dark
        : AppTheme.light;
    return TokenScope(
      tokens: palette.toDesktopTokens(),
      child: const MainPage(),
    );
  },
),
```

### 桥接映射

| AppPalette 字段 | DesktopTokens 字段 | 用途 |
|---|---|---|
| `selectedBg` | `primaryColor` | 选中 / 焦点色 |
| `menuBar` | `backgroundColor` | 窗口背景 |
| `textPrimary` | `foregroundColor` | 主文字 |
| `border` | `borderColor` | 控件边框 |
| `surface` | `surfaceColor` / `cardColor` | 面板背景 |
| `ribbonBar` | `controlColor` | 按钮 / 工具栏 |
| `popupBg` | `popoverColor` | 弹出菜单 |
| `textSecondary` | `mutedForegroundColor` | 次要文字 |
| `tabBar` | `secondaryColor` | 标签栏 |
| `gutterBg` | `mutedColor` | 行号槽 |
| `textOnSelected` | `accentForegroundColor` | 选中态文字 |

> **新增 DesktopTokens 字段时**：在 `toDesktopTokens()` 中补充映射。

---

## 组件归属规则

### ⚠️ 强制规则：禁止在 daro 中创建自定义 UI 组件

**任何 UI 组件需求，必须遵循以下流程：**

1. **优先检查 `base-ui-flutter`**：查看是否已有该组件（见底部组件速查表）
2. **存在 → 直接使用**：从 `base_ui_flutter` barrel 导入使用
3. **不存在 → 在 `base-ui-flutter` 中创建通用组件**：按下方步骤新增，然后在 daro 中引入
4. **绝对禁止**：在 `lib/widgets/` 或任何 daro 业务代码中自造 UI 组件（如 `_FormField`、`_CloseButton`、`_ApplyButton` 等）

> **违规示例**：主题定制弹窗中曾出现 `_ApplyButton` 自定义组件，应直接使用 base-ui 的 `Button`。

### ⚠️ 强制规则：禁止直接使用 Material 组件

**所有 UI 控件必须使用 `base_ui_flutter` 封装组件，禁止直接使用 Flutter Material 控件**（`ElevatedButton`、`TextField`、`Checkbox`、`Dialog`、`SnackBar`、`Tooltip`、`InkWell`、`Material` 等）。

| 场景 | ✅ 使用 base-ui 组件 | ❌ 禁止的 Material 组件 |
|---|---|---|
| 按钮 | `Button` | `ElevatedButton` / `TextButton` / `FilledButton` / `IconButton` |
| 输入 | `Input` | `TextField` / `TextFormField` |
| 勾选 | `CheckBox` | `Checkbox` / `Switch` / `CheckboxListTile` |
| 下拉 | `ComboBox` | `DropdownButton` / `DropdownMenu` |
| 弹窗 / 提示 | `MessageBox` / `Popover` / `Toast` | `Dialog` / `AlertDialog` / `SnackBar` |
| 分组 | `GroupBox` | `Card` / `ExpansionTile` |
| 标签页 | `TabControl` | `TabBar` / `TabBarView` |
| 点击反馈 | `GestureDetector` / `Listener` / `MouseRegion` | `InkWell` / `InkResponse` |

> **base-ui 未提供的组件**：一律按上述流程在 `base-ui-flutter` 中创建通用组件，禁止直接用 Material 组件替代。
> **布局基础组件**（`Row`、`Column`、`Stack`、`Container`、`ListView`、`GridView`、`CustomPaint` 等）不属于 Material 控件，可正常使用。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpringHgui/daro](https://github.com/SpringHgui/daro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
