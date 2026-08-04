---
trigger: always_on
description: - **技术栈**: Tauri 2.x (Rust) + 原生 HTML/CSS/JS（无框架）
---

# 发票酱 — Agent 指南

## 项目概览

- **版本**: v2.1.2
- **技术栈**: Tauri 2.x (Rust) + 原生 HTML/CSS/JS（无框架）
- **前端**: `src/{index.html, styles.css, ocr.js, layout.js, print.js, app.js}`
- **后端**: `src-tauri/src/{main.rs, lib.rs, pdf_engine.rs, pdfium_print.rs}`
- **OFD/XML 解析**: `src-tauri/invoice-engine/` — 独立 crate（v2.0.6 从 ofd-engine 更名，v2.0.7 整合 XML 数电票）
- **双版本**: 轻量版 / OCR版（含 PP-OCRv5）

## 常用命令

```bash
npm run dev             # 轻量版开发
npm run dev:ocr         # OCR 版开发
npm run build           # 轻量版构建
npm run build:ocr       # OCR 版构建
npm run build:all       # 全量构建，产物输出到 dist/
npm run bump <版本号>    # 同步版本号到 Cargo.toml + tauri.conf.json
```

- **版本号数据源**: `package.json` 是唯一数据源
- **编译缓存**: 只改 HTML/JS/CSS 不会触发重编译，需改 Rust 文件才会完整重编译
- **CI/CD**: GitHub Actions，push tag `v*` 触发

### IPC 异步化 (async + spawn_blocking)

所有 CPU 密集型后端命令必须用 `async fn` + `spawn_blocking` 包装，防止 IPC 消息泵饥饿导致 `ERR_CONNECTION_REFUSED`。

- `render_pdf_pages` / `render_pdf_pages_pdfium` / `extract_pdf_text` / `extract_pdf_texts` 均已异步化
- `spawn_blocking` 将计算移到线程池，IPC 线程可继续处理消息
- 非 `async fn` 的同步命令会阻塞 IPC 线程

---

## 架构要点

### PDF 生成双管道

首选 **lopdf 直通管道**（矢量无损）→ 失败时自动回退 **printpdf 渲染管道**

- `generate_pdf_from_layout()` 入口
- lopdf 直通: `can_passthrough_pdf()` 判断 → `extract_page_as_form_xobject()` → JPEG DCTDecode 嵌入
- 打印四模式: PDF阅读器模式(默认) / 弹窗确认 / 静默打印PDFium(推荐) / 静默打印SumatraPDF
- **PDF阅读器模式已知限制**: 通过 `ShellExecuteW` 委托系统默认 PDF 阅读器打印，`printto` 动词能否指定打印机取决于阅读器实现（Edge/Chrome 内置查看器不支持），多数情况下 fallback 到 `print` 动词使用默认打印机，**无法可靠控制打印机选择**

### PDF 渲染双引擎 (v1.9.10+)

首选 **WinRT PDF**（系统组件）→ 失败时自动回退 **PDFium 渲染**

- 启动检测: `check_winrt_pdf_available()` 创建临时 PDF 测试 WinRT `PdfDocument` API
- WinRT 渲染: `render_pdf_pages()` — `windows::Data::Pdf::PdfDocument` + `StorageFile`
- PDFium 渲染: `render_pdf_pages_pdfium()` — `FPDF_LoadMemDocument` + `FPDF_RenderPageBitmap` → PNG
- 前端 fallback 链: `_winrtPdfAvailable` 标志 → WinRT 失败自动切换 PDFium
- PDFium 位图渲染: `pdfium_print::render_pdf_to_images()` — BGRA→RGBA 转换 + PNG 编码

### 预览与打印 DPI 分离 (v1.10.5)

预览和打印使用不同的 DPI 和图片格式，兼顾速度与质量：

- **预览 DPI**: `PDF_PREVIEW_DPI = 150`（屏幕显示足够清晰，是打印 DPI 的一半）
- **打印/保存 DPI**: `PDF_RENDER_DPI = 300`（高质量输出，不变）
- **预览格式**: JPEG（quality 80%），文件体积比 PNG 小 60-80%
- **打印格式**: PDF 直通管道输出矢量 PDF，不受预览分辨率影响
- `RenderedPage.format` 字段：前端据此判断图片格式（`"png"` 或 `"jpeg"`）
- **移除预览时的自适应 DPI 缩放**：自适应缩放仅用于打印质量输出

### 发票字段提取

**路径优先级**: PDF文字层 > OFD XML > XML 数电票 > OCR

- **发票类型检测**: `_detectInvoiceType()` — nontax(优先级最高) > vat > ticket > ride > unknown
- **金额三阶段**: 含税价 → 数学验证配对 → 区域解析
- **中文大写兜底**: `parseChineseNumeral()` — 阿拉伯金额因字体/编码丢失时的 fallback
- **OCR 跳过条件**: `_pdfTextExtracted && sellerName && amountTax > 0`

### 购销方识别优化 (v2.1.1)

修复偶发的「购买方识别为销售方」问题，采用表头锚点 + 交叉验证双保险。

- **表头锚点法** `_determineLabelSide(label, words)`：用「购买方」/「销售方」表头词的 x 坐标作为区域锚点，替代固定 0.5 边界
  - 支持融合词（"购买方"）和 CJK 拆字序列（购+买+方）两种检测方式
  - 双表头：label 归属距离更近的一侧；单表头：以表头 ±0.15 为判定区间；无表头：fallback 到 0.5
- **动态边界** `_getSideBoundary(words)`：双表头中点 / 单表头 ±0.25 / 无表头 0.5
  - 词收集过滤器和信用代码分类统一使用动态边界，保持与 label 分类一致
- **性能缓存** `_headerCache`：按 words 数组引用缓存表头位置，避免紧密循环（信用代码排除、词收集）中重复 O(n) 扫描
- **交叉验证** `_crossValidateBuyerSeller(result, words)`：在 `_extractByText` return 前执行
  - Rule 1：buyerName === sellerName → 清空 sellerName（同名几乎必为识别错误）
  - Rule 2：sellerName.nx < buyerName.nx - 0.15 → 交换（位置反了）
  - Rule 3：sellerCreditCode.nx < buyerCreditCode.nx - 0.15 → 交换
  - Rule 4：sellerCreditCode 同侧的 buyerName 实为销售方 → 迁移
- **影响范围**：`_extractNamesByCoords` / `_extractByText` / `extractByCoordinates` 中所有固定 0.5 边界判定统一改为动态边界；`_extractSeller` 兜底函数的 0.45 宽松阈值保持不变

### 字段提取准确性修复 (v2.1.2)

针对 CJK 拆字格式（dzcp/iloveofd）下的字段提取问题，新增多个兜底策略。

- **信用代码 CC5 兜底**：买方代码被拆成单字 word 时（`9132020013590404` + `X` + `W`），normText 的 `\n` 破坏正则连续匹配。拼接全文 word（无分隔符）匹配独立 18 位代码，排除已找到的 seller 和 invoiceNo
- **Method2 正则优化**：`[0-9\s]` → `[0-9A-Z\s]` 支持字母夹在数字中间的合并格式；新增 15/18 位长度校验
- **名称括号保留**：检测到括号紧跟匹配后缀时不拆分，保留「XXX（分公司）」完整结构
- **CJK 拆字日期合并**：Pattern5 匹配连续 6 词「年/MM/月/DD/日」序列，合并为 YYYY-MM-DD
- **_cleanName 日期清理**：清理 `YYYY年MM月DD日`、`YYYY-MM-DD`、`YYYY/MM/DD` 等格式碎片
- **列表高亮修复**：`syncActiveFileFromPage` 改为先检查 `_activeFileIdx` 是否在当前页范围内，避免覆盖用户点击

### XML 数电票解析 (v2.0.7+)

`invoice-engine::parse_xml_invoice()` 解析独立 XML 数电票文件，提取结构化发票数据。

- **格式**: 纯结构化数据（`<EInvoice>` 根元素），**无版式/排版信息**，不可渲染票面
- **用途**: 文件列表展示、金额统计、汇总表导出、批量重命名
- **不参与排版打印**: `getActiveFiles()` 过滤 `_xmlInvoice` 标记，`getFileIndex()` 返回 null
- **字段提取**: `parse_xml_invoice_fields()` — 字符串匹配提取标签内容，比事件解析更可靠
- **提取字段**: 发票号码/日期/销售方/购买方/金额/发票类型
- **前端标记**: `fileObj._xmlInvoice = true`，无 `previewUrl`/`ow`/`oh`
- **文件列表**: 显示 XML 占位符 + 发票尾号，而非图片缩略图

### 文件列表记忆 (v2.0.7)

可选功能，启动时自动恢复上次打开的文件列表。

- **开关**: `S.feat.fileListMemory`，设置面板「记忆发票列表」，默认关闭
- **恢复机制**: `restoreFiles()` 启动时批量恢复文件路径 → `check_path_exists` 校验存在性
- **标志保护**: `_isRestoringFiles` 标志阻止恢复期间触发 OCR 自动识别
- **轻量设计**: 仅记忆文件路径（不保存金额/OCR 数据），与设置持久化分离
- **路径校验**: 启动时验证文件存在性，自动跳过已删除文件

### 打印状态追踪 (v2.0.7)

追踪发票是否已打印，支持过滤和持久化。

- **三种过滤**: 侧边栏顶部「全部/未打印/已打印」`.print-filter-bar` 过滤按钮组
- **自动标记**: 四种打印模式成功后自动 `markFilesAsPrinted()` → 绿色 ✓ 标识
- **持久化**: `_printedMap` 始终保存到 localStorage，不受功能开关影响
- **迁移**: `clearAll()` / `executeRename()` / `resetSettings()` 均正确迁移打印状态 key

### 版本号显示与检查更新 (v2.1.0)

利用 GitHub Release 作为更新源，启动时自动检查 + 手动触发检查双模式。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erma0/fapiao-print](https://github.com/erma0/fapiao-print) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
