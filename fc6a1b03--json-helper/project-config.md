---
trigger: always_on
description: **Json Helper** 是一个面向 IntelliJ IDEA 的 JSON 效率插件，提供强大的 JSON 数据操作和转换能力。
---

# Json Helper - AI Agent Guide

## Project Overview

**Json Helper** 是一个面向 IntelliJ IDEA 的 JSON 效率插件，提供强大的 JSON 数据操作和转换能力。

**项目元数据：**

| 属性          | 值                                       |
|-------------|-----------------------------------------|
| Group ID    | `com.acme`                              |
| Artifact ID | `json-helper`                           |
| 插件 ID       | `com.acme.json.helper`                  |
| 作者          | 拒绝者                                     |
| 许可证         | MIT                                     |
| GitHub      | https://github.com/fc6a1b03/json-helper |

**核心功能：**

- JSON 编辑、格式化、压缩、修复、转义与反转义
- 从 JSON 生成 Java Class / Record
- 从 Java 类字段复制 JSON 结构
- JsonPath / JMESPath 查询与树形浏览
- URL、JWT、本地文件路径、Web 路径自动解析为 JSON
- JSON 与 XML / YAML / TOML / Properties / CSV / XLSX / Base64 / URL Params 互转
- Search Everywhere 集成：项目搜索与 HTTP 请求文件搜索
- 代码截图复制

## Technology Stack

- **语言**: Java 25
- **构建工具**: Gradle 9.4.1 + IntelliJ Platform Gradle Plugin 2.13.1
- **目标平台**: IntelliJ IDEA 2026.1 (Build 261+)
- **UI 框架**: IntelliJ Platform UI (Swing-based)

### 依赖版本

| 依赖                   | 版本     | 用途           |
|----------------------|--------|--------------|
| IntelliJ Platform    | 2026.1 | IDE 集成       |
| Hutool               | 5.8.44 | 工具库          |
| Jackson              | 3.1.0  | JSON/数据格式处理  |
| Fastjson2            | 2.0.61 | JSON 解析      |
| Auth0 JWT            | 4.5.1  | JWT Token 解析 |
| Apache POI           | 5.5.1  | Excel 文件支持   |
| json-repair          | 0.4.0  | 畸形 JSON 修复   |
| JToon                | 1.0.9  | TOON 格式处理    |
| Apache Commons Lang3 | 3.20.0 | 通用工具         |

### 捆绑插件依赖

- `org.toml.lang` - TOML 语言支持
- `com.intellij.java` - Java 语言支持
- `com.intellij.gradle` - Gradle 支持
- `com.intellij.properties` - Properties 文件支持
- `com.intellij.modules.json` - JSON 模块
- `org.jetbrains.plugins.yaml` - YAML 支持

## Project Structure

```
src/main/
├── java/com/acme/json/helper/
│   ├── common/                    # 通用工具类和枚举
│   │   ├── enums/
│   │   │   ├── AnyFile.java       # 支持的文件类型枚举
│   │   │   └── SupportedLanguages.java
│   │   ├── ActionEventCheck.java  # 动作事件检查（sealed interface）
│   │   ├── Clipboard.java         # 剪贴板操作
│   │   ├── CollectionTypeHandler.java
│   │   ├── TemporalTypeHandler.java
│   │   └── UastSupported.java     # UAST (Universal AST) 支持
│   ├── core/                      # 核心功能模块
│   │   ├── editor/                # 编辑器集成
│   │   │   ├── FileDropHandler.java
│   │   │   ├── JsonEditorPushProvider.java
│   │   │   └── record/EditorState.java
│   │   ├── json/                  # JSON 处理
│   │   │   ├── JsonCompressor.java
│   │   │   ├── JsonEscaper.java
│   │   │   ├── JsonFormatter.java
│   │   │   ├── JsonOperation.java      # Sealed interface
│   │   │   ├── JsonRepairer.java
│   │   │   ├── JsonSearchEngine.java
│   │   │   └── JsonUnEscaper.java
│   │   ├── notice/
│   │   │   └── Notifier.java      # 通知系统
│   │   ├── parser/                # 解析逻辑
│   │   │   ├── AnyParser.java     # 自动格式检测
│   │   │   ├── ClassParser.java   # Java 类解析
│   │   │   ├── JsonNodeParser.java
│   │   │   ├── JsonParser.java    # JSON 转换分发
│   │   │   ├── JwtParser.java
│   │   │   ├── PathParser.java
│   │   │   ├── TypeResolver.java
│   │   │   └── converter/         # 格式转换器
│   │   │       ├── Base64Converter.java
│   │   │       ├── ClassConverter.java
│   │   │       ├── CsvConverter.java
│   │   │       ├── DataFormatConverter.java
│   │   │       ├── JavaStructure.java
│   │   │       ├── PropertiesConverter.java
│   │   │       ├── RecordConverter.java
│   │   │       ├── TableStructure.java
│   │   │       ├── TomlConverter.java
│   │   │       ├── ToonConverter.java
│   │   │       ├── UrlParamsConverter.java
│   │   │       ├── XlsxConverter.java
│   │   │       ├── XmlConverter.java
│   │   │       └── YamlConverter.java
│   │   ├── screenshot/
│   │   │   └── CodeScreenshotSupplier.java
│   │   ├── search/                # 搜索功能
│   │   │   ├── cache/SearchCache.java
│   │   │   ├── HttpRequestSearch.java
│   │   │   ├── ProjectSearch.java
│   │   │   └── item/
│   │   │       ├── HttpRequestItem.java
│   │   │       └── ProjectNavigationItem.java
│   │   └── settings/              # 插件设置
│   │       ├── PluginSettings.java
│   │       └── PluginSettingsState.java
│   └── ui/                        # 用户界面
│       ├── MainToolWindowFactory.java
│       ├── action/                # 动作
│       │   ├── json/
│       │   │   ├── CopyJsonAction.java
│       │   │   ├── CreateClassFromJsonAction.java
│       │   │   └── JsonHelperAction.java
│       │   ├── screenshot/
│       │   │   └── CodeScreenshot.java
│       │   └── search/
│       │       ├── HttpRequestSearchAction.java
│       │       └── ProjectSearchAction.java
│       ├── dialog/                # 对话框
│       │   ├── ConvertAnyDialog.java
│       │   └── CreateClassDialog.java
│       ├── editor/                # 编辑器组件
│       │   ├── CustomizeEditorFactory.java
│       │   └── Editor.java        # Sealed interface
│       ├── panel/                 # UI 面板
│       │   ├── JsonTreePanel.java
│       │   └── MainPanel.java
│       └── search/                # 搜索 UI
│           ├── HttpRequestSearchFactory.java
│           └── ProjectSearchFactory.java
└── resources/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fc6a1b03/json-helper](https://github.com/fc6a1b03/json-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
