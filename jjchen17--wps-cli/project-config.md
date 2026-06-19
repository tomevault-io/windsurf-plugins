---
trigger: always_on
description: 通过 WPS Office COM 自动化操作 Word/Excel/PPT/PDF。76 条 CLI 命令，100% 格式保真，--json 输出适配 AI Agent。触发词：wps, word, excel, ppt, pdf, 文档, 表格, 演示, 公文, 模板, 办公自动化, WPS Office, docx, xlsx, pptx, 合并PDF, 格式转换。When you need to: create/edit/programmatically manipulate Office documents on Windows via WPS COM automation.
---

# 设计参考: iOfficeAI/OfficeCLI (Apache 2.0, https://github.com/iOfficeAI/OfficeCLI)

# WPS CLI Skills

> AI Agent 教学文件：让 Claude Code、Cursor 等 AI 工具理解并正确使用 wps-cli 的全部能力。
> 完整 skill 包位于 `skills/wps-cli/` 目录，包含模块化参考文档。

## 概述

**wps-cli** 是一个通过 COM 自动化驱动 WPS Office 桌面端的命令行工具。它允许 AI Agent 在 Windows 环境下以编程方式操作 Word、Excel、PowerPoint 和 PDF 文档，无需人工交互。

### 核心能力

- **Word (Writer)**：创建、读取、修改、查找替换、表格操作、图片插入、页面布局、导出 PDF
- **Excel (Calc)**：单元格读写、公式设置、区域操作、图表创建、排序筛选、工作表管理
- **PowerPoint (Impress)**：幻灯片管理、文本读写、图片插入、备注管理、导出 PDF
- **PDF**：元信息读取、合并、拆分、提取页面、水印添加
- **格式转换**：Word/Excel/PPT 之间的格式互转（含 PDF）

### 限制

| 限制项 | 说明 |
|--------|------|
| 平台 | **仅 Windows**（依赖 COM 和 win32com） |
| WPS 版本 | 需要 WPS Office 2019 或更高版本 |
| 并发 | 单进程模式，同一会话对象不应跨线程并发使用 |
| 网络 | 无需网络连接，所有操作在本地完成 |

## 架构

### 三层架构

```
CLI 层 (typer)
    │
    ▼
Service 层 (业务逻辑)
    │
    ▼
COM Backend (win32com)
    │
    ▼
WPS Office 桌面应用
```

### 关键组件

- **SessionManager**：管理 WPS 进程生命周期（启动/关闭/复用）
- **WriterService**：Word 文档的所有操作逻辑
- **CalcService**：Excel 工作簿的所有操作逻辑（含公式安全防护）
- **ImpressService**：PPT 演示文稿的所有操作逻辑
- **PdfService**：PDF 处理逻辑（基于 Writer 的 PDF 导出能力）
- **ExportService**：跨应用格式转换
- **StyleEngine**：预设样式管理（公文格式、报告格式等）

### 安全设计

- **COM 宏自动执行禁用**：`AutomationSecurity = msoAutomationSecurityForceDisable`，防止恶意宏
- **公式注入防护**：`CalcService.cell_formula` 禁止 `SHELL()`, `DDE()`, `HYPERLINK()`, `WEBSERVICE()` 等危险函数
- **路径遍历防护**：所有输入路径经过符号链接、UNC、目录遍历检查
- **长度限制**：文本替换上限 1000 字符，水印文字上限 100 字符，glob 结果上限 200

## 命令速查

> 所有命令均支持 `--json` / `-j` 标志输出 JSON 格式，便于 AI 解析。
> 全局选项：`--help` 查看帮助，`--json` 输出 JSON。

### 环境诊断

```bash
wps doctor              # 诊断环境：检查 Python、pywin32、WPS 组件
wps doctor --report     # 输出脱敏 Markdown 诊断报告（适合粘贴到 GitHub Issue）
wps version             # 输出版本号
```

### Word 操作 (writer)

| 命令 | 说明 | 示例 |
|------|------|------|
| `wps writer info <file>` | 文档元信息（页数/字数/字符数/段落数/作者） | `wps writer info report.docx --json` |
| `wps writer count <file>` | 字数统计 | `wps writer count report.docx --json` |
| `wps writer replace <file> <old> <new>` | 查找替换文本 | `wps writer replace doc.docx "旧" "新"` |
| `wps writer replace <file> <old> <new> -w` | 通配符替换 | `wps writer replace doc.docx "张?" "李?" -w` |
| `wps writer table-get <file> -i 1` | 读取第 1 个表格 | `wps writer table-get data.docx -i 1 --json` |
| `wps writer table-insert <file> -r 3 -c 4 -d '[["A","B"]]'` | 插入 3x4 表格 | `wps writer table-insert doc.docx -r 3 -c 4 -d '[[...]]'` |
| `wps writer image-insert <file> -i <image>` | 插入图片 | `wps writer image-insert doc.docx -i photo.png` |
| `wps writer image-insert <file> -i <image> -w 200 -h 150` | 插入图片并指定尺寸 | `wps writer image-insert doc.docx -i photo.png -w 200 -h 150` |
| `wps writer page-setup <file>` | 页面布局设置 | `wps writer page-setup doc.docx --width 210 --height 297` |
| `wps writer export-pdf <file> -o <output>` | 导出 PDF | `wps writer export-pdf doc.docx -o output.pdf` |
| `wps writer style-apply <file> <preset>` | 应用预设样式 | `wps writer style-apply doc.docx "公文正文"` |
| `wps writer style-apply <file> "" -l` | 列出可用样式预设 | `wps writer style-apply doc.docx "" -l` |
| `wps writer new -o <path>` | 新建空白文档 | `wps writer new -o blank.docx` |

**可用样式预设**：公文标题、公文一级标题、公文二级标题、公文正文、报告标题、报告正文

### Excel 操作 (calc)

| 命令 | 说明 | 示例 |
|------|------|------|
| `wps calc info <file>` | 工作簿元信息 | `wps calc info data.xlsx --json` |
| `wps calc sheet-list <file>` | 列出所有工作表 | `wps calc sheet-list data.xlsx --json` |
| `wps calc cell-get <file> <ref>` | 读取单元格值 | `wps calc cell-get data.xlsx A1 --json` |
| `wps calc cell-get <file> <ref> -s <sheet>` | 指定工作表读取 | `wps calc cell-get data.xlsx B3 -s Sheet2 --json` |
| `wps calc cell-set <file> <ref> <value>` | 设置单元格值 | `wps calc cell-set data.xlsx A1 "Hello"` |
| `wps calc cell-range <file> <ref>` | 读取区域 | `wps calc cell-range data.xlsx A1:D10 --json` |
| `wps calc cell-formula <file> <ref> <formula>` | 设置公式 | `wps calc cell-formula data.xlsx B10 "=SUM(B1:B9)"` |
| `wps calc chart-create <file> -d <range> -t <type>` | 创建图表 | `wps calc chart-create data.xlsx -d A1:C10 -t pie --title "销售占比"` |
| `wps calc sort <file> -b <col> --order <asc/desc>` | 排序 | `wps calc sort data.xlsx -b A --order desc` |
| `wps calc export-csv <file> -o <output>` | 导出 CSV | `wps calc export-csv data.xlsx -o data.csv` |
| `wps calc new -o <path>` | 新建工作簿 | `wps calc new -o blank.xlsx` |

**图表类型**：`bar`（柱状图）、`line`（折线图）、`pie`（饼图）、`scatter`（散点图）、`area`（面积图）

### PPT 操作 (impress)

| 命令 | 说明 | 示例 |
|------|------|------|
| `wps impress info <file>` | 演示文稿元信息 | `wps impress info slides.pptx --json` |
| `wps impress slide-list <file>` | 列出所有幻灯片 | `wps impress slide-list slides.pptx --json` |
| `wps impress slide-add <file> -l 1 -t "标题"` | 新增幻灯片 | `wps impress slide-add slides.pptx -l 1 -t "新页面"` |
| `wps impress slide-delete <file> <index>` | 删除幻灯片 | `wps impress slide-delete slides.pptx 3` |
| `wps impress text-get <file> -s <idx>` | 获取幻灯片文本 | `wps impress text-get slides.pptx -s 2 --json` |
| `wps impress text-set <file> -s <idx> -p title -t "标题"` | 设置占位符文本 | `wps impress text-set slides.pptx -s 1 -p title -t "新标题"` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjchen17/wps-cli](https://github.com/jjchen17/wps-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
