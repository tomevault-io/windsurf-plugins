---
trigger: always_on
description: 给后续开发（含 AI 辅助）的**统一约束**。写新功能、改 UI 前先读这一份；细节的「为什么」在对应文件的 KDoc 里，这里只列**必守的规则**与去哪找。
---

# Inkwell 开发规范

给后续开发（含 AI 辅助）的**统一约束**。写新功能、改 UI 前先读这一份；细节的「为什么」在对应文件的 KDoc 里，这里只列**必守的规则**与去哪找。

原则：**改动要读起来像周围的代码** —— 匹配既有的注释密度、命名与惯用法。这个项目习惯用注释讲清「为什么这样、不这样会怎样」，新代码请延续。

---

## 模块结构与边界

| 模块 | 内容 | 依赖约束 |
|---|---|---|
| `core/` | 书源引擎、规则解析、分页模型无关的解析、备份合并、Legado 兼容 | **纯 JVM**，不依赖 Android/Compose，必须能脱离 Android 单测 |
| `reader/` | 分页、渲染、翻页、测量、阅读器 API | Android 库，但排版核心尽量可测（见 `reader/src/test`） |
| `app/` | UI（Compose）、ViewModel、数据层（Room/DataStore/repo）、更新、DI | 依赖上面两个 |

- **可测优先**：核心逻辑（解析、分页、书源规则、备份合并）下沉到纯 JVM，脱离 Android 也能单测。新逻辑优先放能测的地方。
- `reader` 的公开 API 用 ARGB `Long` 表颜色，**不要**让 reader API 反向依赖 Compose 类型。

## 构建与测试

```bash
export JAVA_HOME=/opt/java/jdk-21.0.11+10   # 需 JDK 21
./gradlew :core:test :reader:test :app:testDebugUnitTest   # 单测
./gradlew assembleDebug                                     # 整包
```

提交前至少跑通 `assembleDebug` + 三个模块单测。

**动了 `:baselineprofile` 就再跑一条**：`./gradlew :baselineprofile:compileNonMinifiedReleaseKotlin`。它不在上面任何一条链里 —— 那个模块只在生成 profile 时才编译，所以连语法错都能一路溜到 CI，代价是一次起模拟器的六十分钟。（真栽过：KDoc 里写了 `androidx/navigation/` 加通配星号，那个 `/*` 序列在 Kotlin 里会开一层**嵌套**块注释，KDoc 结尾的 `*/` 只关掉内层，外层一路吃到文件末尾。）

---

## UI 统一层（硬规则）

所有 UI 尺寸、动效、颜色、圆角、排版**一律走令牌 / 封装组件**，不写裸值。加新令牌前先问一句：「现有的哪个不够用？」

### 尺寸 → `app/.../ui/components/Dimens.kt`

全部落在 **4dp 栅格**。常用：`gapXS`(4)/`gapS`(8)/`gapM`(12)/`gapL`(16)/`gapXL`(24)/`gapXXL`(32)；页面 `screenPadding`(20)；设置行 `rowHorizontal`(20)/`rowVertical`(10)；内容列表行 `listHorizontal`(16)/`listVertical`(4)；`sectionHeaderTop`(16)、`chipSpacing`(4)、`buttonMinHeight`(40)；图标 `iconSm`(18)/`iconMd`(24)/`iconLg`(32)/`iconXL`(48)；`touchTarget`(48)；`buttonSpinner`(18)；封面 `coverThumbWidth/Height`、详情 `coverDetailWidth/Height`、书架网格 `bookshelfGridMin`；输入高 `searchFieldHeight`/`compactFieldHeight`（均为 40）。**禁止**在页面里写 `16.dp`、`10.dp` 之类；同类元素跨页面尺寸必须一致。

**Expressive 管形态与动效，Compact 管密度：** 继续用 `MaterialExpressiveTheme`、带 `shapes()` 的按钮/Chip/图标按钮重载、ListItem 交互重载；默认行内边距用 `ContentListDefaults.CompactPadding`，带封面/大图标预览的行才显式 `ComfortablePadding`。别为了「收矮」退回普通 `MaterialTheme` 或外层另搓 `Surface` 冒充列表行。

### 动效 → `app/.../ui/components/Motion.kt`

**组件、浮层与页面进退都以 `MaterialTheme.motionScheme` 为唯一来源。** 优先用 `topBarEnter/Exit`、`bottomBarEnter/Exit`、`scrimEnter/Exit`、`expandEnter/Exit`、`pagePushTransform` / `pagePopTransform`；需要裸 spec（`animateItem`、`AnimatedContent`）时读令牌，**别裸写 `tween(300)`**。
- **spatial / effects 分工**（M3 约定，混了会难看）：位移与尺寸用 `defaultSpatialSpec()`，纯视觉属性（alpha、颜色）用 `defaultEffectsSpec()`。
- **「退场比入场快」**：帮手里入场 `default*`、退场 `fast*`。
- **系统「移除动画」**：`InkwellTheme` 把 `motionScheme` 换成 `InstantMotionScheme`；帮手里不必再判 `animationsEnabled()`。`animateItem` 仍可传 `null` 省插值；reader 翻页在主题外。
- **页面进退**：Expressive 共享轴 X —— 新页整屏从右滑入、旧页左让约 1/4（返回镜像）；**不要加 fade**（半透明叠字会退化成渐隐渐显）。
- **阅读器开合仍硬编码 tween**（`READER_ENTER_MS` / `READER_EXIT_MS`）：与 splash 窗口咬合，别改成令牌 spring。这是动效层唯一刻意例外。

### 主题入口 → `MaterialExpressiveTheme`

App 走 **M3 Expressive**。主题入口有两个 —— 全局 `InkwellTheme`（`ui/theme/Theme.kt`）与阅读页浮层的 `ReaderThemeScope` —— **两个都必须**是 `MaterialExpressiveTheme`。任一处退回普通 `MaterialTheme` 都会把 `LocalUsingExpressiveTheme` 关掉，那一片区域的组件就悄悄变回非 Expressive 形态（编译不报错，只是长得不一样）。

`motionScheme` 这个参数**必须显式传**，别省：它默认 `null`，而 `null` 的含义是「沿用外层主题的方案」而非「填 expressive」—— 根节点没有外层，省掉就退成 `MotionScheme.standard()`，结果是组件形态换了、动效没换，编译与单测都看不出来。

代价记清楚：Expressive 只在 material3 **1.5.0-alpha** 通道上（1.4.0-beta01 把这批 API 整批摘出了稳定线），所以 compose 依赖用的是 **`compose-bom-alpha`**，整个 Compose 栈都在预发布线。升级 BOM 前先读 release notes 的 breaking change 段，别盲升。

### 圆角 → `MaterialTheme.shapes`

刻度在 `Theme.kt`：`extraSmall`(4)/`small`(8)/`medium`(12)/`large`(16)/`extraLarge`(28，对齐 M3 Dialog)。用 `MaterialTheme.shapes.medium` 等，**不要**裸写 `RoundedCornerShape(12.dp)`。

### 颜色 → `MaterialTheme.colorScheme` 语义令牌

页面颜色一律走语义令牌（`primary`/`surface`/`onSurface`/`surfaceContainer*`/`error`…），**不写十六进制、不写 `Color.Gray`**。配色由 `AppThemes.kt` 从「强调色 + 背景色」推导整套（含 `surfaceContainer*` 全槽位）。
- 正文性小字用 `onSurfaceVariant`（对比度达标），**不要**用 `outline`（浅色下仅约 3.9:1，达不到 WCAG 4.5:1）。`outline` 只作边框/分隔线。
- **唯一例外**：阅读器**内容区**颜色（纸色/字色）走 `ReaderTheme` / `ReaderThemeScope`（`app/.../ui/reader/ReaderThemeScope.kt`），这是有意的独立主题；但阅读器 UI 的尺寸/动效仍走上面的令牌。阅读器里的浮层用 `ReaderThemeScope` 包裹，让 Chip/Slider/分隔线自动协调，别挨个传色。

### 排版 → `MaterialTheme.typography`

用角色（`display/headline/title/body/label`），别裸写 `fontSize = 14.sp`。正文 ≥ `bodySmall`(12sp)。

### 复用封装组件（别手搓）

有封装就用封装，不要复制粘贴裸 M3 组件：

| 需求 | 用这个（`app/.../ui/components/`） |
|---|---|
| 按钮（带 loading，不撑大） | `PrimaryButton` / `SecondaryButton`（`AppButtons.kt`） |
| 图标按钮 | `AppIconButton`（`AppButtons.kt`）；顶栏返回键一律 `BackButton`。**别裸写 `IconButton`** —— 裸写的落到不带 `shapes` 的旧重载，按下去没有 Expressive 的圆角形变，同一条顶栏上就分成两派手感 |
| 有标题的内容页顶栏 | `AppTopBar` + `rememberAppTopBarScroll()` + `Modifier.topBarScroll(...)`（`AppTopBar.kt`，Expressive `MediumFlexibleTopAppBar`）。**三样必须配齐**：只换组件不接滚动＝白送一条永久变高的顶栏。四类例外仍用经典窄栏，理由写在 `AppTopBar` 的 KDoc 与各页注释里（标题位是交互控件的搜索页/发现页、带多选态的书架与书源管理） |
| 顶栏/工具条搜索框 | `SearchField`；对话框/表单行内 `CompactTextField`（`AppTextField.kt`）。两者是手搓 `BasicTextField`（M3 `TextField` 最小 56dp 塞不下），但配色形状取 `TextFieldDefaults.roundedShape` / `tonalColors()`，别自己发明底色 |
| 多行文本输入 | `CompactTextArea`（同文件）：同一套 tonal 色，高度 `textAreaMinHeight`，圆角走 `shapes.large`。**不要**用 `roundedShape` —— 那是按高度一半算的胶囊，160dp 高的框会变成两头大圆 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radiumCN/inkwell](https://github.com/radiumCN/inkwell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
