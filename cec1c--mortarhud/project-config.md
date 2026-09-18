---
trigger: always_on
description: 给接手这个项目的 AI / 开发者看的。**先读完这份再动手**，能省掉大量重复踩坑。
---

# AGENTS.md —— MortarHUD 项目速读

给接手这个项目的 AI / 开发者看的。**先读完这份再动手**，能省掉大量重复踩坑。

---

## 一、这是什么

《Wardogs》的迫击炮坐标解算外置 HUD。

玩家打开地图、鼠标指向某处时，游戏会在光标旁画出该点的绝对坐标：

```text
y109.78
x98.09
```

程序读出这两行，算出炮位到目标的方位角与距离，用透明置顶、鼠标穿透的 HUD 显示：

```text
AZ  079.0°
RNG 152m
```

**硬约束（改代码时不能破）**：不注入进程、不读写游戏内存、**不模拟键鼠输入**、不联网。
只做两件事：注册全局热键、在按键那一刻截取屏幕上一小块。
完整需求见 `docs/requirements.md`。

---

## 二、环境（最容易卡住的地方）

| 事项 | 说明 |
| --- | --- |
| **.NET 10 SDK** | 装在 `C:\dotnet10`，**不在 PATH 里**。裸 `dotnet` 会解析到 .NET 6（系统 PATH 优先于用户 PATH，且 `C:\Program Files\dotnet` 只有 .NET 6、无写权限）。<br>**一律用 `"C:/dotnet10/dotnet.exe"`。** |
| **PowerShell** | 用 `pwsh`（7.6.6），**不要用 `powershell`**（5.1）。5.1 读无 BOM 的 UTF-8 脚本会按 GBK 解码，中文注释会把语法搞崩。 |
| **Python** | 3.10.11，带 `cv2` / `PIL` / `numpy`，用来分析截图很方便。 |
| **代理** | `127.0.0.1:7890`（已在环境变量里）。GitHub / NuGet 慢时走它。 |

```bash
# 构建
"C:/dotnet10/dotnet.exe" build MortarHUD.sln -c Release

# 测试（当前 202 个，必须全绿）
"C:/dotnet10/dotnet.exe" test MortarHUD.sln -c Release

# 发布（输出到 dist\MortarHUD-next-<模式>\，目录非空会拒绝发布）
publish.cmd              # portable（默认）：单文件压缩 / 约 95MB
publish.cmd folder       # 备选：文件夹布局（287 个文件），启动更快 / 约 227MB
publish.cmd runtime      # 框架依赖：目标机需装 .NET 10 桌面运行时
```

`publish.cmd` 只是 `tools\publish.ps1` 的壳。脚本固定用 `C:\dotnet10\dotnet.exe`，
不再 `where dotnet`（那会解析到 .NET 6）。发布完会校验产物里没有 FFmpeg 残留。

### 启动目录

现在的发布产物是标准扁平布局，**从任意目录启动都可以**——便携版与文件夹版都实测过。
早先的产物用 `libs\` + `additionalProbingPaths` 组织，那种才要求工作目录等于程序目录；
`OrganizeLegacyLayout=false` 之后不再生成这种布局。

---

## 三、诊断工具（排查问题时先想到它们）

```bash
# 启动自检：走完整启动流程，但「不显示窗口、不注册热键、不截取屏幕」
# 构造全部三个窗口，再用随程序发布的实机样本 Models\selftest\roi.png 跑一次端到端识别。
# 识别结果不对会非 0 退出——不会出现「识别挂了但自检仍然绿」。
dist\MortarHUD\MortarHUD.exe --selftest

# 设置窗口离屏渲染成 PNG（窗口不显示、不抢焦点）
dist\MortarHUD\MortarHUD.exe --screenshot <输出目录>

# 追加 --expanded：把所有折叠项展开后再渲染。
# 这条很重要 —— 折叠项展开后的排版本来只能靠人工看，有了它就能自动核对了。
dist\MortarHUD\MortarHUD.exe --screenshot <输出目录> --expanded

# 追加 --compact：用 920x680 渲染（默认 1040x760）
dist\MortarHUD\MortarHUD.exe --screenshot <输出目录> --compact

# OCR 基准测试，会生成 docs\ocr-benchmark.md
# 必须用 dotnet run —— 它会先把工作目录切到项目目录。
# 直接以仓库根为工作目录去跑 bin 里的 dll，OpenCV 原生库会加载失败
# （DllNotFoundException: OpenCvSharpExtern），因为它的解析依赖工作目录。
"C:/dotnet10/dotnet.exe" run --project tools/MortarHUD.Benchmark -c Release

# 把每条流水线的二值化结果 dump 成 PNG —— 调 OCR 时唯一靠谱的手段
"C:/dotnet10/dotnet.exe" run --project tools/MortarHUD.Benchmark -c Release -- --dump <输出目录>

# 从实机截图重新学习字形模板库
"C:/dotnet10/dotnet.exe" run --project tools/MortarHUD.Benchmark -c Release -- --gen-templates
```

日志：`%AppData%\MortarHUD\Logs\yyyy-MM-dd.log`（输入层、热键、采集、OCR 明细都在里面）
Debug 转储：`%AppData%\MortarHUD\Debug\`（需在设置里开启）

---

## 四、架构

```
src/
├─ MortarHUD.Core/              纯计算，无 Windows / UI 依赖
│  ├─ Models/                   MapCoordinate、MortarSolution、CoordinateOcrResult
│  ├─ Ballistics/               距离 + 方位角（atan2(东, 北)，顺序别写反）
│  ├─ Parsing/                  x/y 解析 + OCR 字符修正
│  ├─ Validation/               范围 / 置信度校验
│  ├─ Session/                  状态机、HUD 排版、Debug 排版、操作调度（LatestOperationRunner）
│  ├─ Configuration/            设置模型 + 持久化 + schema 迁移
│  ├─ Themes/                   主题模型 / 内置主题 / 读写
│  └─ Diagnostics/              文件日志
│
├─ MortarHUD.Capture/           截屏 → 预处理 → OCR
│  ├─ ScreenCapture/            IScreenCaptureProvider + GDI 实现 + ROI 计算
│  ├─ ImageProcessing/          Pipeline A / B / C
│  ├─ Diagnostics/              Debug 转储（原始 ROI / 预处理图 / 结果 JSON）
│  └─ Ocr/                      引擎接口、Tesseract、模板匹配、交叉验证编排
│
├─ MortarHUD.Platform.Windows/  所有 Win32 互操作
│  ├─ Hotkeys/                  RegisterHotKey + Raw Input（含按下/松开去重）
│  ├─ Mouse/                    光标位置、前台窗口与归位判断（CaptureContext）
│  ├─ NativeMethods/            Win32 / Gdi32 / RawInput
│  ├─ WindowStyles/             Overlay 窗口样式、显示器信息
│  ├─ Dpi/                      Per-Monitor V2
│  └─ Startup/                  开机自启
│
└─ MortarHUD.App/               WPF
   ├─ Views/                    Overlay、Debug 面板、设置窗口（三页）、HudRenderer、ColorEditor
   ├─ ViewModels/               SettingsViewModel
   ├─ Services/                 HudController
   ├─ Tray/                     系统托盘
   ├─ Assets/                   应用图标（EXE / 窗口 / 托盘共用同一份 .ico）
   └─ Models/                   随程序发布的 OCR 资源（tessdata + 字形库 + selftest 样本）
```

**核心原则**：`Capture ≠ OCR ≠ Parser ≠ Calculator ≠ Overlay`，每层可独立替换与测试。

### OCR 是怎么工作的

1. 光标附近切 ROI（默认 `offset(-15,-90)` `150x140`，**由三张实机截图实测反推**，不是拍脑袋）
2. 用**两条独立流水线**（A：对比度拉伸 + Otsu；C：顶帽 + 双门限）分别二值化
3. 各自跑 Tesseract，各自解析
4. **交叉验证**：有**任何**分歧就整体判失败（`PIPELINE_DISAGREEMENT`）——
   不投票取多数，也不挑一个用。多加一条相似流水线不能把一条不同的读数「投」下去
5. 严格解析：必须两位小数、不许歧义、不许猜小数点
6. **最后**才用综合置信度（`0.55 + 0.25×一致比例 + 0.20×引擎自报`）跟 `MinimumConfidence` 比。
   单条流水线的原始置信度**不**参与门槛判断——实测 Tesseract 对读对了的坐标也会给 0.00

**单条流水线只有 2/3 正确率，且错的不是同一例**；交叉验证后 3/3。这是整个 OCR 设计的立足点。

---

## 五、当前未决问题（明天从这里开始）

### 🔴 1. OCR 间歇性失败（最优先）

**现象**：同一个坐标、连续点多次，有概率失败。**用户已确认不是遮挡导致的。**

**当前缓解**：采集失败自动重试 3 次（间隔 120ms），
并且整轮采集已经串行化 + 可取消——迟到的识别结果不会再覆盖新状态。

**推测根因**：读数处在二值化临界点上。光标差 1px → 抗锯齿变一点 → Otsu 阈值一切就翻。
属于固定阈值方案的固有抖动，重试只是缓解。

**根治需要数据**，现在还没有。采集入口已经做好了：

```
设置 → 诊断 → 「收集接下来 10 次采集」
→ 进游戏复现失败
→ 取 %AppData%\MortarHUD\Debug\ 里最新的那批文件：
     *_raw.png        实际截到的画面
     *_processed.png  二值化结果  ← 关键，能直接看出是字被削断还是压根没读出来
     *_result.json    每条流水线的原始文本与耗时
```

这个按钮只收集指定次数就自动停，不用一直开着全局 Debug 开关。

看到 `_processed.png` 才能判断是：
- 笔画被阈值削断 → 调 Pipeline A/C 的参数

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cec1c/MortarHUD](https://github.com/Cec1c/MortarHUD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
