---
trigger: always_on
description: macOS 原生 SEG-Y / SGY 地震数据查看器，对标 Windows 的 SeiSee，核心是超大文件（10 GB 级、数十万道）的**即时显示**。纯 Swift，零第三方依赖。界面支持中文（简体）与 English，首次启动跟随系统语言，可在 View → Language 切换（即时生效、持久记忆），并内置双语使用说明（⌘?）。剖面横向排列支持三种：按道号 / 按偏移距（有符号）/ 按偏移距（绝对值）。
---

# SeisView — 开发与维护手册

macOS 原生 SEG-Y / SGY 地震数据查看器，对标 Windows 的 SeiSee，核心是超大文件（10 GB 级、数十万道）的**即时显示**。纯 Swift，零第三方依赖。界面支持中文（简体）与 English，首次启动跟随系统语言，可在 View → Language 切换（即时生效、持久记忆），并内置双语使用说明（⌘?）。剖面横向排列支持三种：按道号 / 按偏移距（有符号）/ 按偏移距（绝对值）。

---

## ⚠️ 先读：开源与保密约束（最高优先级）

本仓库已公开到 GitHub（MIT License，**仅个人署名**）。因此**仓库内容（含提交信息与全部历史）绝对禁止出现**：

- 单位 / 研究院 / 国企名称
- 真实数据文件名与目录名
- 本机绝对路径（含用户主目录）

**这些关键词的确切清单与「推送前扫描命令」保存在本地项目 memory（`seisview-github-repo`）里，刻意不写进本仓库**——写进来等于再次泄露。每次改动后、`git push` 前，务必按 memory 里的命令全量扫描（工作树 + 全部历史），输出为空才推。

其他约定：

- `docs/` 已加入 `.gitignore`：设计文档 / 实现计划（含实测性能数据、训练口径）刻意不公开，本地保留。
- 大文件黄金测试 / 性能回归读 `SEGY_BIG_FILE` 环境变量，未设置则**跳过**。**任何代码都不得硬编码数据路径。**
- 提交作者统一为个人署名 + 个人邮箱（勿用本机自动生成的 `.local` 假邮箱）。

---

## 架构

```
SegyKit（纯核心，零 UI 依赖，可独立测试）
├── Types.swift        SampleFormat / ByteOrder / Geometry / BinaryHeader / TraceHeader
├── ByteOrder.swift    ByteOrderReader.u16/u32/i32（大/小端读取）
├── IBM.swift          IBM→IEEE 解码（指数查找表）
├── Decoder.swift      按 SampleFormat 解码原始字节 → [Float]
├── SegyFile.swift     打开、头解析、几何推断与校验、假 IBM 自动校正
├── TraceReader.swift  并行 pread + 解码（每线程独立 fd，整道大块读）
├── Decimator.swift    min/max 分箱降采样
├── Gain.swift         百分位 / AGC / 每道 / maxAbs 标定 + GainKind（去载荷的种类）
├── Rasterizer.swift   振幅 → CGImage（灰度/红白蓝/红白黑/棕白黑 4 配色，256×3 LUT）
├── Viewport.swift     纯值类型视口状态 + 平移/缩放/重置/百分比换算 + decodePlan + 中心锚缩放 + 缩放映射 + maxTraceSpan 上限
├── ScrollMetrics.swift 滚动条滑块几何（长度/偏移/像素↔索引反算）
├── ShotIndex.swift    FFID 炮索引（抽样 + 二分）
├── TraceOrder.swift  剖面排列方式（byTrace / byOffset / byOffsetAbs）
└── OffsetIndex.swift 炮内 offset 排序置换（有符号/绝对值两套 perm）+ 每炮起始位置

Localization（纯文案库，依赖 SegyKit，被 SeisView 与 SegyKitTests 共用）
├── Lang.swift         语言状态（系统检测 + 用户覆盖）
├── StringKey.swift    S 枚举（92 个界面文案 key）
├── Tables.swift       zh/en 两张文案表 + string/format
├── ErrorText.swift    SegyError → 本地化用户文案
├── MenuTitles.swift   菜单标题中英反查（纯函数）
└── HelpContent.swift  双语使用说明九章（结构化数据，中英严格对应）

SeisView（AppKit + SwiftUI）
├── SeisViewApp.swift  入口 + 工具栏（增益/百分比/调色板/局部放大/道头开关/重置/对齐/炮导航）+ ZoomBar 缩放条 + LineSlider 单线滑块 + onOpenURL + ContentView/StatusBar
├── DocumentModel.swift 已开文件 + 视口状态 + 渲染管线 + showHeaderInspector/zoomRectMode/zoomToRect（@MainActor ObservableObject）
├── SectionView.swift   剖面显示（NSViewRepresentable + 双向滚轮平移/捏合/点击选道/右键框选缩放/光标）
├── ScrollBar.swift     自绘水平/垂直滚动条 + ScrolledSection 包裹布局
├── HeaderInspector.swift 道头表格（字节位置 + 值）
├── L10n.swift           语言状态（UserDefaults 持久化）+ errorMessage 渲染
├── MainMenuLocalizer.swift 遍历 NSApp.mainMenu 重命名（系统项按 selector、其余按标题反查）
├── HelpWindow.swift     双语使用说明窗口（HelpView + HelpMenuButton，⌘?）
└── CompareLayout.swift  多文件并排（HSplitView 可拖动分隔）

SegyKitTests（自定义 harness 可执行目标，非 XCTest）
├── Harness.swift      @MainActor 断言工具（check/checkClose/checkRel/finish）
└── main.swift         全部测试入口
```

关键设计：渲染是纯函数 `(SegyFile, Viewport) → CGImage`；`Viewport` 是 `Equatable + Sendable` 纯值类型。

---

## 构建 / 测试 / 运行 / 发布

```bash
swift build                       # 构建 4 个 target（SegyKit / Localization / SegyKitTests / SeisView）
swift run SeisView                # 直接运行
swift run SegyKitTests            # 跑测试（当前 222 断言，非零退出码 = 失败）

./scripts/make_app.sh             # 快速打当前架构的 .app
./scripts/release.sh [版本号]      # 通用二进制 + dmg + zip（产出在 dist/）

# 真实文件黄金测试 + 性能回归（未设则跳过）：
SEGY_BIG_FILE=/path/to/big.segy swift run SegyKitTests

# 重新生成图标：
swift scripts/make_icon.swift && iconutil -c icns Resources/SeisView.iconset -o Resources/SeisView.icns
```

环境：macOS 13+，Swift 6.0+。**只需 Command Line Tools，不需要完整 Xcode**（纯 Swift + SwiftPM、零第三方依赖，无 Xcode 工程与资源 bundle，构建/测试/打包全走命令行；`notarytool`/`stapler`/`hdiutil` 都可用）。

---

## 关键技术事实（改代码前必读）

### SEG-Y 字节偏移（二进制头 slice `raw[0]` = 文件字节 3200，即 1-indexed 字节 − 3201）

| 字段 | 1-indexed | raw 下标 |
|---|---|---|
| dt（微秒） | 3217–3218 | raw[16..17] |
| ns（每道采样数） | 3221–3222 | raw[20..21] |
| 数据格式码 | 3225–3226 | raw[24..25] |
| SEG-Y 版本 | 3501–3502 | raw[300..301] |
| 定长道标志 | 3503–3504 | raw[302..303] |
| 扩展文本头数 | 3505–3506 | raw[304..305] |

道头（`p` 指向 240 字节道头起始）：道序 = u32(p+0)、FFID = u32(p+8)、CDP = u32(p+20)、偏移距 = **i32**(p+36)（有符号）、ns = u16(p+114)、dt = u16(p+116)。

> 坑：`raw[300..305]` 曾错写成 `raw[100..105]`（3501−3201=300 算成 100），导致扩展头文件几何错乱。这是最容易再犯的错误。

> 坑：偏移距（offset，字节 37–40）是**有符号 int32**。若用 u32 读，炮点一侧的负 offset 会变成 ~42 亿，排序时排到炮内最后、剖面呈「先升后跳」的假反转。这是 0.3.0 修过的真实 bug，别改回 u32。

### IBM 浮点解码

`value = sign × mantissa × 2^(4E − 280)`，其中 E = 7 位指数、mantissa = 24 位尾数；mantissa==0 → 0。用 128 项 `expTable` 预计算 2^(4e−280)。

### 格式码与字节序

支持 1=IBM32、2=int32、3=int16、5=IEEE32、8=int8。格式码高字节 0x8000 表示 rev2 小端。**假 IBM 真 IEEE**：声明 IBM 但实际存 IEEE 时，首道样本按两种方式各解一遍、比较「有限且在 [1e-6,1e6] 内」的比例，IEEE 明显更合理（>3×）则自动校正，存 `formatWasCorrected`。

### ns 冲突启发式

二进制头 ns 与首道道头 ns 不一致时，**取与文件大小整除吻合的那个**，都吻合/都不吻合时以二进制头为准（SEG-Y rev1 权威字段）。真实文件道头 ns 可能不可靠。

### min/max 分箱是「正确性」不是「性能」

4000 采样点压进 800 px 时若隔点取样会混叠、剖面严重失真。每个像素 bin 必须取 (min, max)（Decimator），Rasterizer 用**主振幅** `abs(mx)>=abs(mn) ? mx : mn` 而非中点。

### 变长道检测

道数 = (文件大小 − 首道偏移) / 道长，**必须整除**；除不尽 → 明确报错（修掉 SeiSee 静默错乱的缺陷）。`extOffset` 之前必须 `guard size >= extOffset`（防 UInt64 下溢）。

---

## Swift 6 严格并发 / 测试约定


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZDDWLIG/SeisView](https://github.com/ZDDWLIG/SeisView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
