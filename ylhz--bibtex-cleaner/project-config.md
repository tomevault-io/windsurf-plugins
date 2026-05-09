---
trigger: always_on
description: > **最后更新时间：** 2026-01-21 18:59
---

# BibTeX Pro Formatter - AI 助手开发指南

> **最后更新时间：** 2026-01-21 18:59
> 
> 本文件为 Claude Code 和 GitHub Copilot 提供代码开发指导。

## 项目概述

BibTeX Pro Formatter 是一个 **100% 客户端 Web 应用**，用于清理、标准化和格式化学术论文的 BibTeX 条目。主要功能包括：
- DBLP 集成与自动学习场所缩写
- 多格式引用导出（BibTeX、MLA、GB/T 7714）
- 批量处理模式与警告检测
- 批量验证模式 - 验证论文是否在 DBLP 中真实存在

**核心理念："永不猜测"** - 工具使用严格的三层解析策略：
1. 自定义/学习规则（从本地存储精确匹配或正则匹配）
2. DBLP 提示（用户点击搜索结果时）
3. 回退方案（保留原始全名并警告用户）

## 技术栈
- **纯 JavaScript (ES6 模块)** - 无框架（React/Vue/Angular）
- **原生 DOM 操作** - 直接使用 querySelector/createElement
- **LocalStorage** - 持久化配置
- **无构建工具** - 直接在浏览器中运行

## 架构

### 核心文件

```
js/
├── main.js           - 单条模式、DBLP 搜索、UI 编排
├── batch-mode.js     - 批量模式逻辑、条目管理
├── processor.js      - BibTeX 解析、场所映射、引用键生成
├── config.js         - ConfigManager 单例、常量
├── utils.js          - 文本处理工具（作者名、标题）
├── venue_data.js     - 默认场所映射规则（基于正则）
├── warning-system.js - 警告检测和过滤
├── ai-detector.js    - AI 生成内容检测
├── dblp-validator.js - DBLP 批量验证与匹配算法
└── formatters/       - 输出格式化器（BibTeX、MLA、GB/T 7714）
    ├── bibtex.js
    ├── mla.js
    └── gbt7714.js
```

### 数据流

1. **输入来源：**
   - 用户粘贴原始 BibTeX → `dom.input`
   - 用户搜索 DBLP → `performSearch()` → 点击结果 → 自动填充输入
   - 批量模式 → 解析多个条目 → 逐个处理

2. **处理流程：**
   ```
   main.js:runConversion() / batch-mode.js:parseBatchInput()
     ↓
   processor.js:processEntries()
     ├─ parseRawBibtex()           [用正则提取条目]
     ├─ parseMappingRules()        [加载场所映射规则]
     ├─ 场所解析：
     │   1. 检查 customRules（LocalStorage 学习规则）
     │   2. 检查 mappingRules（设置中的正则模式）
     │   3. 使用 hintVenue（来自 DBLP 点击）
     │   4. 回退：保留原始 + 警告
     ├─ 清理 DBLP 伪影           [移除四位数消歧编号]
     └─ 生成引用键               [格式：[Auth][Year][Title]_[Venue]]
     ↓
   formatters/[format].js          [输出为 BibTeX/MLA/GB/T 7714]
     ↓
   main.js:renderOutput() / batch-mode.js:renderEntriesList()
   ```

3. **自动学习系统：**
   - 用户点击 DBLP 结果时：`LAST_CLICKED_VENUE_HINT` 捕获场所
   - 获取后：`ConfigManager.addCustomRule(fullName, abbr)` 保存到 localStorage
   - 未来转换：`customRules[fullName]` 提供即时匹配

### 状态管理

**单条模式：**
- `CURRENT_DATA` - 已处理的 BibTeX 数据
- `CURRENT_WARNINGS` - 验证警告
- `CURRENT_TAB` - 当前输出格式

**批量模式：**
- `BatchModeState.entries` - 所有条目数组
- `BatchModeState.isActive` - 模式切换
- `BatchModeState.currentFilter` - 当前筛选器（'all', 'warnings', 'ignored', 'ai', 'confirmed'）
- `BatchModeState.ignoredWarnings` - 已忽略的警告类型

**LocalStorage（通过 ConfigManager）：**
- `bib-fields` - 保留哪些 BibTeX 字段
- `bib-format` - 引用键格式模板
- `bib-mappings` - 用户编辑的映射规则（文本）
- `bib-venue-mode` - 输出模式：`'abbr'` 或 `'full'`
- `bib-keep-original` - 是否保留原始引用键
- `bib-custom-rules` - 自动学习的场所映射（对象）
- `bib-search-mode` - DBLP 获取模式：`'simple'`（元数据）或 `'detailed'`（.bib 文件）
- `bib-ignored-warnings` - 批量模式已忽略警告

## 代码风格与约定

### 语言规范
- **所有面向开发者的文件和注释使用简体中文**
- 包括但不限于：代码注释、文档、README、提交信息
- 用户界面文本根据目标用户语言设置
- 变量名、函数名仍使用英文（遵循 JavaScript 惯例）

### JavaScript
- 使用 ES6 模块，显式导入/导出
- 回调和事件处理器使用箭头函数
- 异步操作使用 async/await
- 适当使用对象解构
- HTML 生成使用模板字面量

### 命名约定
- 函数：`camelCase`（如 `handleSearchDBLP`、`updateStats`）
- 常量：`UPPER_SNAKE_CASE`（如 `CURRENT_DATA`、`BATCH_MODE_STATE`）
- DOM 引用：缓存在 `dom` 对象中
- 事件处理器：以 `handle` 开头（如 `handleRecheck`、`handleIgnoreEntry`）

## 关键实现规则

### 1. DOM 操作
- **缓存 DOM 引用**到模块级 `dom` 对象
- 使用 `querySelector` / `querySelectorAll` - 避免 jQuery 模式
- 动态内容使用 `innerHTML` 配合模板字面量
- 始终使用 `escapeHtml()` 工具转义用户输入

### 2. 状态管理
- 单条模式：`CURRENT_DATA`、`CURRENT_WARNINGS`、`CURRENT_TAB`
- 批量模式：`BatchModeState` 对象包含条目数组
- 不要直接修改状态 - 使用更新函数
- 状态变更后调用 `updateStats()` 和 `renderEntriesList()`

### 3. BibTeX 处理
- 使用 `parseRawBibtex()` 解析（processor.js 中基于正则的解析器）
- 使用 `processEntries()` 处理（场所映射、字段过滤、ID 生成）
- 使用 `toBibTeX()` / `toMLA()` / `toGBT()` 格式化输出
- 尽可能保留原始结构

### 4. 警告系统
- 使用 `detectWarnings(entry, processingResult)` 检测
- 存储在 `entry.warnings` 数组中
- 支持忽略标志：`entry.isIgnored = true`
- 警告解决时自动清除：检查 `warnings.length === 0`

### 5. 批量模式特定规则
- 条目具有状态：`warnings`、`isIgnored`、`isAISuspected`
- 筛选器：'all'、'warnings'、'ignored'、'ai'、'confirmed'
- **"已确认"** = 无警告且未忽略
- **"已忽略"** = `isIgnored = true`（保留警告以便编辑）
- 编辑后点击"🔄 检查"重新验证
- 警告解决后自动清除 `isIgnored` 标志

### 6. DBLP 集成

**快速模式（默认，`simple`）：**
- 仅使用 DBLP 搜索 API 元数据
- 在 `generateBibFromJSON()` 中从 JSON 生成 BibTeX
- 无额外网络请求
- 立即学习场所规则

**精准模式（`detailed`）：**
- 通过 `fetchAndFillBibtex()` 获取官方 .bib 文件
- 尝试：主 URL → 代理 → HTML 抓取回退
- 使用 `parseRawBibtex()` 正则解析
- 从 `booktitle`/`journal` → DBLP 场所学习映射

### 7. ConfigManager 使用
- 获取字段：`ConfigManager.getFields()`（不是 getFieldsConfig）
- 获取格式：`ConfigManager.getFormat()`（不是 getIDFormat）
- 获取场所模式：`ConfigManager.getVenueMode()`（'abbr' | 'full'）
- 自定义规则：`ConfigManager.getCustomRules()` 返回对象
- 保留原始：`ConfigManager.getKeepOriginal()` 返回布尔值

### 8. 场所映射边界情况

1. **NIPS → NeurIPS 修正**（main.js、processor.js 的 generateBibFromJSON）
2. **Findings 检测**：
   - 检查 `ee` 字段是否包含 'findings' 关键字
   - 防止主会议规则覆盖（例如 "ACL" 规则不会匹配 "ACL Findings"）
3. **词边界保护**：
   - 自动用 `\b(?:...)\b` 包裹正则模式以防止误匹配
   - "RAL" 不会匹配 "Neural" 内部

## 常用模式

### 在批量模式中添加按钮操作
```javascript
// 1. HTML 生成（createEntryCard）
<button data-action="my-action" data-entry-index="${index}">标签</button>

// 2. 事件绑定（bindCardEvents）
card.querySelectorAll('[data-action]').forEach(btn => {
  btn.addEventListener('click', () => {
    handleEntryAction(btn.dataset.action, index, entry);
  });
});

// 3. 操作处理器（handleEntryAction）
case 'my-action':
  handleMyAction(index);
  break;
```

### 编辑后更新条目（批量模式）
```javascript
// 1. 解析新 BibTeX

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ylhz/bibtex-cleaner](https://github.com/ylhz/bibtex-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
