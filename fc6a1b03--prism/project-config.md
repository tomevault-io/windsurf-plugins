---
trigger: always_on
description: 本文件面向 AI 编码代理，假设读者对本项目一无所知。所有内容均已与实际代码核对。
---

# Prism - AI Agent 指南

本文件面向 AI 编码代理，假设读者对本项目一无所知。所有内容均已与实际代码核对。

## 项目概览

**Prism**（原名 Json Helper）是一个面向 IntelliJ IDEA 的效率插件：JSON 侧提供数据编辑、查询、生成与多格式互转能力；增强侧集成代码地图（minimap）、彩虹括号与变量高亮、项目树压缩包浏览与文件注释、代码截图、项目 / HTTP / 端口搜索等编辑器增强功能。

**项目元数据：**

| 属性        | 值                                              |
|-------------|-------------------------------------------------|
| Group ID    | `com.acme`                                      |
| Artifact ID | `prism`                                   |
| 插件 ID     | `com.acme.prism`                          |
| 作者        | 拒绝者                                          |
| 当前版本    | `0.18.8`（定义在 `settings.gradle` 的版本表中） |
| 许可证      | MIT                                             |
| GitHub      | https://github.com/fc6a1b03/prism         |

**核心功能：**

- JSON 编辑、格式化、压缩、转义与反转义
- 从 JSON 生成 Java Class / Record
- 从 Java 类字段复制 JSON 结构
- JsonPath / JMESPath 查询与树形浏览
- URL、JWT、本地文件路径、Web 路径自动解析为 JSON
- JSON 与 XML / YAML / TOML / Properties / CSV / XLSX / Base64 / URL Params 互转
- Search Everywhere 集成：项目搜索、HTTP 请求文件搜索、端口搜索、压缩包内容搜索
- 项目树中将压缩包（zip / 7z / jar / war / ear / tar / tar.gz / tgz / tar.bz2 / tbz2 / tar.xz / txz / gz / bz2 /
  xz）作为目录展开浏览
- 项目树文件信息：文件名右侧显示头部注释摘要与修改时间
- 编辑器增强：彩虹括号配对高亮、彩虹变量高亮、颜色字面量 Gutter 色块、代码地图（minimap）
- 代码截图复制

## 技术栈

- **语言**: Java 25（toolchain 与 `options.release` 均取自版本表 `jvm`）
- **构建工具**: Gradle 9.6.1 + IntelliJ Platform Gradle Plugin 2.18.1
- **目标平台**: IntelliJ IDEA 2026.2（sinceBuild = 262，由 IGP 2.14+ 按目标平台 major build 默认生成，无需显式声明）
- **UI 框架**: IntelliJ Platform UI（Swing-based）
- **字符编码**: 全项目 UTF-8

### 依赖版本

版本统一在 `settings.gradle` 的 `gradle.ext.versions` 表中维护：

| 依赖                    | 版本   | 用途                     |
|-------------------------|--------|--------------------------|
| IntelliJ Platform       | 2026.2 | IDE 集成                 |
| Hutool                  | 5.8.47 | 工具库（core + http）    |
| Jackson                 | 3.2.1  | JSON/数据格式处理（BOM） |
| Fastjson2               | 2.0.62 | JSON 解析与校验          |
| Auth0 JWT               | 4.5.2  | JWT Token 解析           |
| Apache POI              | 5.5.1  | Excel 文件支持           |
| Apache Commons Compress | 1.28.0 | 压缩包格式支持           |
| JUnit                   | 6.1.2  | 单元测试（BOM，test 域） |
| Apache Commons Lang3    | 3.20.0 | 通用工具（强制统一版本） |

### 捆绑插件依赖（`build.gradle` 与 `plugin.xml` 中声明）

- `org.toml.lang` - TOML 语言支持
- `com.intellij.java` - Java 语言支持
- `com.intellij.gradle` - Gradle 支持
- `com.intellij.properties` - Properties 文件支持
- `com.intellij.modules.json` - JSON 模块
- `com.intellij.modules.vcs` - VCS 模块（Git 行状态）
- `org.jetbrains.plugins.yaml` - YAML 支持

## 项目结构

```
src/main/
├── java/com/acme/json/helper/
│   ├── common/                    # 通用工具类和枚举
│   │   ├── enums/
│   │   │   ├── AnyFile.java       # 支持的文件类型枚举 (JSON, XML, YAML, TOML, CLASS, RECORD, CSV, XLSX 等)
│   │   │   └── SupportedLanguages.java  # 支持的语言枚举
│   │   ├── ActionEventCheck.java  # 动作事件检查（sealed interface Check）
│   │   ├── Clipboard.java         # 剪贴板操作
│   │   ├── CollectionTypeHandler.java   # 集合类型处理
│   │   ├── TemporalTypeHandler.java     # 时间类型处理
│   │   └── UastSupported.java     # UAST (Universal AST) 支持
│   ├── core/                      # 核心功能模块
│   │   ├── archive/               # 压缩包格式识别、索引、树节点、搜索、打开器、图标
│   │   │   ├── ArchiveFormats.java      # 压缩包格式枚举与条目读取
│   │   │   ├── ArchiveIndex.java        # 压缩包索引
│   │   │   ├── ArchiveContentIndex.java # 压缩包内容索引（供搜索）
│   │   │   ├── ArchiveTreeProvider.java # 项目树压缩包节点提供
│   │   │   ├── ArchiveIconProvider.java # 压缩包图标提供
│   │   │   ├── ArchiveOpener.java       # 压缩包条目打开器
│   │   │   ├── ArchiveSearch.java       # 压缩包内容搜索
│   │   │   └── ArchiveEntryNode.java    # 压缩包条目树节点
│   │   ├── editor/                # 编辑器集成
│   │   │   ├── FileDropHandler.java         # 文件拖放处理
│   │   │   ├── JsonEditorPushProvider.java  # 编辑器推送提供者
│   │   │   └── record/EditorState.java      # 编辑器状态记录 (Base64 编码/解码)
│   │   ├── fileinfo/              # 项目树文件信息
│   │   │   ├── FileCommentExtractor.java    # 文件头部注释提取
│   │   │   ├── FileInfoNodeDecorator.java   # 项目树节点装饰
│   │   │   ├── FileInfoCacheService.java    # 文件信息缓存服务
│   │   │   └── FileInfoDisplay.java         # 文件信息展示
│   │   ├── json/                  # JSON 处理
│   │   │   ├── JsonOperation.java       # JSON 操作接口 (sealed interface)
│   │   │   ├── JsonCompressor.java      # JSON 压缩
│   │   │   ├── JsonEscaper.java         # JSON 转义
│   │   │   ├── JsonUnEscaper.java       # JSON 反转义
│   │   │   ├── JsonFormatter.java       # JSON 格式化
│   │   │   └── JsonSearchEngine.java    # JSON 搜索引擎 (JsonPath/JMESPath)
│   │   ├── minimap/               # 代码地图（编辑器缩略图）
│   │   │   ├── MinimapEditorFactoryListener.java  # 编辑器挂载监听器
│   │   │   ├── MinimapPanel.java / MinimapView.java / MinimapRenderer.java
│   │   │   ├── MinimapGeometry.java / MinimapScrollState.java
│   │   │   └── CharacterWeights.java              # 字符油墨权重
│   │   ├── notice/
│   │   │   └── Notifier.java      # 线程安全通知系统
│   │   ├── parser/                # 解析逻辑
│   │   │   ├── AnyParser.java     # 自动格式检测 (TOML/YAML/Properties/CSV)
│   │   │   ├── ClassParser.java   # Java 类解析 (PSI-based)
│   │   │   ├── JsonNodeParser.java      # JSON 节点解析
│   │   │   ├── JsonParser.java    # JSON 转换分发器（EnumMap 注册表）
│   │   │   ├── JwtParser.java     # JWT Token 解析

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fc6a1b03/prism](https://github.com/fc6a1b03/prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
