---
trigger: always_on
description: - 当前文件 `\newcommand` / `\def`、未保存 `.sty` 缓冲区、只失效变化的依赖。
---

# Silk Math Preview 协作规范

## 2026-08-23 边写边加宏/大文件热路径（0.1.76）

- 当前文件 `\newcommand` / `\def`、未保存 `.sty` 缓冲区、只失效变化的依赖。
- 公式打字只看光标处控制词，上方已写完的 `\\newcommand` 不再全文失效。
- 流水线：24 files / 259 tests；覆盖率 stmts/branch/lines `93.45%/87.53%/96.10%`。
  main bundle 204,665 B。cold p50/p95 `130.1/142.9 ms`，warm p50/p95
  `12.5/21.1 ms`，scanner p95 `0.77 ms`，idle restart 通过。
- 产物正式版 `silk-math-preview-0.1.76.vsix` 1,226,235 bytes，SHA-256
  `FC2B054113BD38C7519D4DCF68ADA9FBB8CEAE3CD6A735E67F834FC83B934131`；
  测试版 `silk-math-preview-test-0.1.76.vsix` 1,226,367 bytes，SHA-256
  `D8ABD2B10F91862F6F0F7DFB04EB8B0102CA518AEC2817364A6039C82D451808`。

## 2026-08-22 边写边加宏/宏包，大 sty 也要丝滑

- 用户 `\newcommand` / `\usepackage` / 本地 `.sty/.cls` **本来就能进预览**；缺口是未保存缓冲区、
  任意文件改动清空全部解析缓存、大文件 `split('')` 复制、超过 2 MB 静默丢掉。
- 定义解析：`maskTeXComments` 无 `%` 时零拷贝；`parseTeXSource` 一次 mask 扫声明和依赖。
  4000 条宏 p95 内解析。`.sty/.cls` 上限 8 MB，超限写 limitation，不再默默跳过。
- 工作区：优先读已打开文档（未保存的 sty 也能用）；只失效变化的那一个来源；
  peek 在失效后仍保留上一份快照。磁盘 watcher / 文本变更会让预览 48 ms 内重算定义。
- Windows 宏包文件名大小写不敏感；Linux 保持大小写敏感。不加载 CTAN，不执行 expl3。
- 公式打字只看光标处控制词，上方已写完的 `\\newcommand` 不再触发全文失效。
- 流水线：24 files / 259 tests。main bundle 204,665 B（硬门 204,800），不含 MathJax。
  cold p50/p95 `127.8/143.5 ms`，warm p50/p95 `13.3/23.0 ms`，scanner p95 `1.13 ms`，
  idle restart 通过。快照 prelude 已接到 MathJax 实画。
- 本轮未发版、未打 VSIX。

## 2026-08-22 商店搜索与介绍（0.1.75）

- 名字仍是 Silk Math Preview。`description` / `keywords`（最多 30 个）/ `categories`
  改成 Visualization · Notebooks · Education，覆盖 latex、mathjax、jupyter、
  equation、ocr 等搜索词。README 写明 Marketplace 链接。
- GitHub 仓库描述和 topics 同步；主页项目/动态另改 PersonalWebsite-source。
- 流水线：23 files / 235 tests。cold p50/p95 `126.2/135.1 ms`，warm p50/p95
  `13.1/20.3 ms`，scanner p95 `0.87 ms`。main bundle 202,297 B。
- 产物正式版 `silk-math-preview-0.1.75.vsix` 1,225,453 bytes，SHA-256
  `BECA01CB76BE860F74A701EC7A948D0A431110258E7D39F5778CABBD956E14E0`；
  测试版 `silk-math-preview-test-0.1.75.vsix` 1,225,584 bytes，SHA-256
  `00ECF954FF689F7D3512104D63D45B09FCF9E75724727A45CBC0246E8CC86C47`。

## 2026-08-22 常见语言与介绍页（0.1.74）

- 商店只渲染 README.md：页内 `#english` 起 11 种常见语言，示意图仍 4 张。
  不要另开 README.zh-CN.md。命令面板标题仍中英并列，避免测试通道再拆 nls。
- 界面文案在 `src/core/uiLocale.ts`，按 `vscode.env.language` 选；`zh-tw/hk/mo/hant`
  走繁体，其他 `zh*` 走简体，对不上回退英文。菜单、状态栏、OCR 宿主和 OCR
  面板共用这一份；OCR bundle 只收当前语言，不打包整份目录。
- 流水线：23 files / 234 tests；覆盖率 stmts/branch/lines `93.43%/87.51%/96.05%`。
  main bundle 202,297 B。cold p50/p95 `134.2/146.6 ms`，warm p50/p95 `13.0/20.4 ms`，
  scanner p95 `0.83 ms`，idle restart 通过。
- 产物正式版 `silk-math-preview-0.1.74.vsix` 1,223,654 bytes，SHA-256
  `3DF8B2CC823A5E8B4C3C02BAE4DEB8CDC16401B48A3A4BEE01D13DD8B5889B20`；
  测试版 `silk-math-preview-test-0.1.74.vsix` 1,223,795 bytes，SHA-256
  `57BA27B6B34974CC163439BB544377F56E99516BC16BC3E9EAECE9AC0FB8D5AA`。

## 2026-08-22 表格合并单元格（0.1.73）

- `\multicolumn{n}{align}{...}` / `\multirow{n}{*}{...}` 用 class 标跨度，SVG 按
  相邻 mtd/mtr 的 translate 把内容移到合并区域中心。列数仍用 `&` 占位，避免串列。
- 产物正式版 `silk-math-preview-0.1.73.vsix` 1,203,031 bytes，SHA-256
  `E4EB2C807079C62CC94B64BC289E01C835E3B12C41AF70EED3B532BC230D34C1`；
  测试版 `silk-math-preview-test-0.1.73.vsix` 1,203,169 bytes，SHA-256
  `E79996E40E3352C5C1AFF7FD857EB6988282BF5D4A0FF5F1CEF51AEFC899A93F`。

## 2026-08-22 正式包与测试包分开（0.1.72）

- `npm run package:release` → `silkmath.silk-math-preview`，发 Marketplace。
- `npm run package:test` → `silkmath.silk-math-preview-test`，命令 `silkMathTest.*`，
  设置 `silkMathTest.*`，状态栏 Silk Math Test。两份可同时安装。
- 测试清单 `private: true`。CI 仍只把正式包 `vsce publish`。
- 产物正式版 `silk-math-preview-0.1.72.vsix` 1,201,814 bytes，SHA-256
  `179DB3EB764E1AEFEC27B352A9708EDD24AD90A2F6FFB8FABAD16F7482471891`；
  测试版 `silk-math-preview-test-0.1.72.vsix` 1,201,950 bytes，SHA-256
  `326EE12C89115ADA8A2A9914B226BF50055DE9A7F66956D76EEBF4A3926691AD`。

## 2026-08-22 inner y 向上配对表首（0.1.71）

- MathJax 表格线在 `scale(1,-1)` 下，属性 min-y 是视口下方。TeX 表首配对
  max inner y，表尾配对 min inner y，朝表内加线。混合边界测试用
  `tableRulesInRootSpace` 在翻转后的根坐标里量，表首双线靠近 A 行。
- 流水线：21 files / 224 tests。cold p50/p95 `145.9/161.6 ms`，warm p50/p95
  `15.2/32.8 ms`，scanner p95 `0.94 ms`，idle restart 通过。main bundle 128,845 B。
  产物 `silk-math-preview-0.1.71.vsix`，`1,201,523` bytes，SHA-256
  `6F0BDB67719A996B28D90901D3C94DFC4F28598838EFBC1DF46F226558B24A6E`，15 个条目。

## 2026-08-22 表首双线必须是两根填充横线（0.1.70）

- 框线 `data-frame` 不是填充 `data-line=h`。表首 count≥2 时从 inner max y
  （SVG min-y）画出两根填充横线，间隙一根线宽。混合边界测试断言
  `data-line=h` 至少两根。
- 流水线：21 files / 224 tests。main bundle 128,845 B。cold p50/p95
  `124.0/132.8 ms`，warm p50/p95 `12.5/20.0 ms`，scanner p95 `0.84 ms`，
  idle restart 通过。产物 `silk-math-preview-0.1.70.vsix`，`1,201,394` bytes，
  SHA-256 `1A8B0B0C06ABCD1D6903AD9F5700C0E2E183116CD3C3A0D985B6B62D01FB438F`，15 个条目。

## 2026-08-22 单侧双横线配对（0.1.69）

- `splitDoubleHlines` 不再把含 0 的 TeX 计数和 min-y 槽位 1:1 对齐。只把
  count≥2 的边界配到对应可见线：表首 min-y，表尾 max-y，行间用内线。
  `{|c|c|}\\hline\\hline A\\\\B` 这类只在一侧双线的表不再跳过或画错边。
- 流水线：21 files / 224 tests；覆盖率 stmts/branch/lines `93.56%/87.81%/96.08%`。
  main bundle 128,845 B。产物 `silk-math-preview-0.1.69.vsix`，`1,201,306` bytes，
  SHA-256 `9D89E2D85668DBF0AC9FEFD7533B8F7F10985ECE1A2EDFE7F402680B1C218556`，15 个条目。

## 2026-08-22 双线表格、鼠标点选、未写完公式与热路径（0.1.68）

- 连续 `\hline\hline`：MathJax 会把同一行边界的第二条覆盖掉。按 array 表达式
  计数在 SVG 里再画一根，间隙取线宽、方向朝表内。列格式 `||` 本来就是两根
  近距竖线，收到大约一根线宽。只配对同向共轴的近距线。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhoujasper/silk-math-preview](https://github.com/zhoujasper/silk-math-preview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
