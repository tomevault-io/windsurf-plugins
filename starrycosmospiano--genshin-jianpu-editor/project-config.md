---
trigger: always_on
description: 简谱（JP-Word / `.jpwabc`）排版与编辑器。这是原 Kotlin/JVM + JavaFX + Skija 桌面应用
---

# jpeditor-web

简谱（JP-Word / `.jpwabc`）排版与编辑器。这是原 Kotlin/JVM + JavaFX + Skija 桌面应用
（仓库根 `../`）向 **Tauri 2 + TypeScript + SVG** 的迁移版。完整方案见
`~/.Codex/plans/abundant-sniffing-dragon.md`。

## 架构决策（已定，勿轻易推翻）

- **渲染用 SVG**（不是 Canvas 2D / CanvasKit）。乐谱页面树（PageItem/Group/GraphicPath/
  GraphicLine/TextFrame）直接映射到 SVG DOM。
- **"在哪测量就在哪绘制"**：排版期的文本宽度/紧包围盒用浏览器的 `getBBox` /
  `getComputedTextLength`（见 `src/common/measure.ts`），与 SVG 渲染同一引擎，天然一致；
  **不需要原生字体测量**，不需要 CanvasKit，不需要 DPI 位图缩放。
  - `Path.computeTightBounds()` → `pathTightBounds(d)`（临时 `<path>`.getBBox）
  - `font.measureText()` → `measureGlyphText()`（`<text>`.getComputedTextLength）
- **MusicXML 已放弃 JAXB**。普通简谱导入/导出由前端 `src/score/musicxml.ts` 和
  `src/score/musicxml-export.ts` 完成；`src/mixed/` 保留独立的五线谱混排模型。
  `src/score/score.ts` 故意不包含旧 JAXB 风格的 `Score.load/Part.load/...` 方法。
  **IDML 导出已彻底放弃。**
- **逻辑分层**：排版/渲染/模型/编辑/格式转换主要在前端 TS；Rust 承接 Tauri 文件能力、
  SVG/PDF、原生 ONNX OCR、Gemini 命令和 macOS MIDI 播放。

## 命令

```bash
npm run dev            # Vite 开发服务器
npm run build          # tsc 严格检查 + vite 打包
npx tsc --noEmit       # 仅类型检查（CI 用）
npm run check:core     # 格式/排版/钢琴/MIDI/文本谱核心回归
npm run check:all      # 核心回归 + Edge 浏览器端回归
npm run tauri dev      # 跑 Tauri 桌面应用（需 Rust）
cd src-tauri && cargo check   # 仅检查 Rust 侧

# 无头渲染/交互校验（用本地 Edge，免下载 chromium）：
npm run build && node shot.mjs /tmp/out.png            # 截 #score-pane + 诊断
npm run build && node abc-check.mjs                    # ABC→MusicXML 移植回归（见 ABC 节）
npm run build && node abc-shot.mjs <abc> /tmp/abc.png  # 拖入 .abc 端到端渲染核对
```

`shot.mjs` 用 Playwright `channel: "msedge"` 驱动本地 Edge，serve `dist/`，加载后截图并
打印页数/着色 token 数/控制台错误。改了渲染相关代码后用它做回归。
`window.__app`（App 实例）在运行时暴露，便于脚本化测试（如 `__app.setText(...)`）。

## 目录与数据流

```
.jpwabc 文本
  → JpwFile.fromString          src/jpword/jpwfile.ts   分段(.Title/.Voice/.Words/...)
  → ANTLR 词法/语法              src/jpword/parse.ts     复用 Jpwabc.g4 生成的 TS 解析器
  → fromJpw → Score             src/score/jpwimport.ts  + src/score/score.ts (模型)
  → JinpuPainter.resize → 排版   src/layout/painter.ts   + src/layout/layout.ts (引擎)
  → SVG DOM                      painter.renderPage(i)
```

- `src/common/` — `fraction.ts`、`geom.ts`（Point/Rect/Matrix33，含 `toSvg()`）、
  `measure.ts`（SVG 测量基础设施，**核心**）。
- `src/smufl/smufl.ts` — Bravura 元数据加载（`public/redist/bravura_metadata.json`）+
  GlyphCodes。**PUA 码位用 `String.fromCharCode(0x...)`，切勿在源码里写字面 PUA 字符**
  （Write 工具会损坏这些字节）。
- `src/jpword/tokens.ts` — `TokenData` 分词器，仅用于编辑器语法高亮（非语义解析）。
- `src/editor/` — `app.ts`（编辑器↔实时重排↔翻页↔文件 I/O 控制器）、`highlight.ts`
  （CodeMirror 装饰）、`file-format.ts`（共享扩展名注册）、`document-parser.ts`（实时解析边界）、
  `fileio.ts`（UTF-16LE 编解码 + Tauri 运行时探测）。
- `src/bootstrap/` — 默认示例、拖拽和缩放等平台/UI 启动适配层；`main.ts` 只编排启动顺序。
- `src/jpword/parser/` — **ANTLR 生成代码，勿手改**，每个文件首行 `// @ts-nocheck`。

## 与原 Kotlin 的对应

按文件近乎逐行翻译。改行为前先看 `../src/main/kotlin/` 对应文件确认原意：
`layout.kt→layout/layout.ts`、`draw.kt→layout/painter.ts`、`score.kt→score/score.ts`、
`jpw.kt→score/jpwimport.ts`、`jpwfile.kt→jpword/jpwfile.ts`、`skia.kt→common/geom.ts`。
Skija 值类型不可变（offset/inset/union 返回新对象）——TS 端保持同样语义。

## 混排（src/mixed/）的参考源与测试数据

- **`src/mixed/` 移植自 C++ 工程 musicpp，路径 `~/proj/musicpp`**。改混排
  行为前先核对 musicpp 原文（render.ts↔`model/render.cpp`、model.ts↔`model/model.cpp`、
  loader.ts↔`mxml/loader.cpp`、painter.ts↔`util/pao.cpp`）。代码里的 `render.cpp:行号` 注释
  即指该仓库。
- **测试 musicxml 在 `~/Documents/Praise as One/`**（只用其中的 `.musicxml/.xml`，
  忽略目录里其它文件）。部分子目录有同名 `*.pdf`（Sibelius 原始排版）可作 slur/tie/小节线
  对位的视觉基准。无头渲染混排：`node shot.mjs out.png --xml <path>`（`window.__mixedPainter`）。

## 简谱图像识别（OMR，`src/omr/`）

把简谱图片（PNG/JPG/PDF）识别成 MusicXML，再走编辑器现有 `importBytes`→`loadMusicXml` 导入排版。
拖拽入口由 `bootstrap/drag-drop.ts` 分流到 `App.recognizeBytes`；工具栏「识图」用于在识别叠加视图与
普通简谱之间切换。识别核心保留**两种方式**：

**PDF 输入**（拖入 `.pdf`，格式注册见 `editor/file-format.ts`，适配层见 `bootstrap/drag-drop.ts`）
经 `decode.ts` 的 `pdfToImageData` 转位图再走
同一 OMR 管线。用 **pdf.js（`pdfjs-dist`）**：worker 经 Vite `?url` 引入，位图解码器 wasm 目录（jbig2.wasm
兼管 **CCITTFax G4**、openjpeg 管 JPEG2000）在 `public/redist/pdfjs/`，**必须**用 `getDocument({wasmUrl})`
指明——否则内嵌位图（扫描版乐谱多是 1-bit `ImageMask`）会被 pdf.js 静默丢弃、页面只剩矢量文字。**优先直接抽取
内嵌位图**（`largestPageBitmap`：`getOperatorList` 找 `paintImage(Mask)XObject` → `objs.get(id)` 拿解码好的
`ImageBitmap`）而非整页渲染——源本就是二值扫描图，直接贴白底即可（顺带甩掉赞美诗页码/栏目标题等叠加矢量文字，
如「耶稣普治」PDF 顶部的 `055/圣子耶稣`）；纯矢量 PDF（无内嵌图）退回 `page.render` 整页光栅化。多页竖向拼接。

- **`gemini`**：整页交 Antigravity CLI `agy` 让 Gemini 直接转写（真实照片更准）。**仅桌面版**：
  `agy` 是命令行工具，经 Rust `omr_gemini_cmd`（[src-tauri/src/lib.rs](src-tauri/src/lib.rs)，
  `std::process::Command`，stdin 关掉防挂起）调用；浏览器内 `agyAvailable()` 为 false → 报"需桌面版"。
- **`musicpp`**：**完全本地**，浏览器/桌面均可、可离线。`decode.ts`(图→二值) → `jianpu.ts`
  (连通域/几何启发式：数字块拆分、下划线 div、八度点、增时线) → `musicxml.ts`(→partwise)；
  数字/歌词/页眉 OCR 走本地 **PaddleOCR PP-OCRv6_small**（`paddleocr.ts`，onnxruntime-web 浏览器离线推理，
  逐数字格 / 歌词条 rec→CTC），**不经 agy**——整页识别本就是 Gemini 方案在做的事。模型/字典在
  `public/redist/ocr/`（rec onnx `ch_PP-OCRv6_small_rec_infer.onnx` **~21MB** + `ppocrv6_dict.txt` **18708 字**
  + **det onnx ~4.7MB**（DBNet，仍 PP-OCRv4，页眉用；det 头与 rec 无关故可跨版混用）），wasm 运行时在
  `public/redist/ort/`（纯 wasm 单线程，免 COOP/COEP）；`onnxruntime-web/wasm` 子入口避开 26MB 的 jsep 构建。
  旧的 **tesseract.js** 后端（`localocr.ts` + `montage.ts`）保留为 fallback（`localOcrBackend()`）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StarryCosmosPiano/genshin-jianpu-editor](https://github.com/StarryCosmosPiano/genshin-jianpu-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
