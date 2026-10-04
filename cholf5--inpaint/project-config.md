---
trigger: always_on
description: lxfater/inpaint-web 的 C# / .NET 10 + Avalonia 桌面重写：MI-GAN 图片修复 + Real-ESRGAN ×4 高清化 + 导出压缩（PNG/JPEG/WebP），纯本地处理，无服务器、无 JS。
---

# AGENTS.md

lxfater/inpaint-web 的 C# / .NET 10 + Avalonia 桌面重写：MI-GAN 图片修复 + Real-ESRGAN ×4 高清化 + 导出压缩（PNG/JPEG/WebP），纯本地处理，无服务器、无 JS。

## 构建 / 运行

- 需要 .NET 10 SDK。`dotnet build Inpaint.slnx`；`dotnet run --project src/Inpaint.App`。
- 单元测试在 `tests/Inpaint.Tests`（xunit.v3 + Avalonia.Headless），`dotnet test` 运行；覆盖 Core 布局/遮罩转换、Inference 分块语义（`UpscaleEngine.FillTile`/`CopyTileCore` 为 internal，经 `InternalsVisibleTo` 供测试）与 App 层（ViewModel 生命周期、画布指针输入，`[AvaloniaFact]` 走 headless）。**需要 Avalonia 平台的参数化测试必须用 `[AvaloniaTheory]`**——裸 `[Theory]` 不走 headless 引导，`new WriteableBitmap` 直接报 "Unable to locate IPlatformRenderInterface"。不含需要模型文件或 ONNX session 的路径。**没有 .editorconfig / 格式化配置**——除测试外，验证手段就是编译通过加手动运行。CI（ubuntu）会跑同一套测试，两条已踩过的跨平台坑：像素断言不能采文本带（HistoryGraphView 标签 y≈94~106，Linux 次像素 AA/字体回退会给字形边缘染上彩边，采样点须选纯图形空白带）；测试不得假设真实用户 `settings.json` 已存在（全新机器/CI 上没有，快照前判存在性、结束时按原状删除或还原）。
- headless 测试入口 `TestAppBuilder` 必须用 `UseHeadlessDrawing = false` + `.UseSkia()`：headless 自绘位图的 `WriteableBitmap.Lock`/`CopyPixels` 语义与生产 Skia 不一致，会得到假结果。注意 App 命名空间与同名命名空间冲突（`Inpaint.App.App` 需别名）。
- macOS Dock 图标只来自 `.app` bundle 的 Info.plist（`CFBundleIconFile`）或运行时设 `NSApplication`，XAML `Window.Icon`/csproj `ApplicationIcon` 对它无效。开发期 `dotnet run` 是裸进程，由 `MacDockIcon`（libobjc 手发消息，失败静默）在桌面生命周期建立后设内嵌 icns 补上（对 bundle 是覆盖而非幂等）；正式包 `scripts/package-macos.sh [arm64|x64] [--fdd]` 产出 `artifacts/macos/Inpaint.app`（自包含、ad-hoc 签名，版本读 csproj `<Version>`），换 icns 后 Dock 有缓存需 `touch` bundle 或重启 Dock。图标圆角烤在素材里（Apple 模板：1024 画布、824 身、185.4 圆角）：Tahoe 会给 Finder 里直角 icns 自动蒙圆角，但 `setApplicationIconImage` 运行时图标和 Windows `.ico` 都不走蒙版；改图标先改 `app-icon.png` 母版，再用 4x 超采样圆角蒙版出 PNG，icns 走 `iconutil`、ico 走 PIL 多尺寸重生成。
- 发版：`scripts/release.sh x.y.z [--skip-test] [--watch]`——校验（main、工作树干净、三段数字版本、tag 不冲突）→ 本地 `dotnet test` → 改 csproj `<Version>` 提交 → 打 `v` tag push → CI（dotnet-desktop.yml）测试 + macOS/Windows/Linux 打包（Windows 另出 Inno Setup 安装包，共 5 个产物）+ 建 GitHub Release；`--watch` 轮询 CI 并核对 Release 产物（需 `gh` 已登录）。Agent 收到「发版 x.y.z」即跑该脚本（带 `--watch`），成功后向用户汇报 Release 链接与产物清单；失败按脚本 stderr 处理，CI 红则看 Actions 日志，修复后删远端/本地 tag（`git push origin :refs/tags/vX`、`git tag -d vX`）再重跑。tag 与 csproj 版本不一致会被 workflow 拒绝，重发同版本必须先删 tag。

## 目录与分层

- `inpaint-web/`：原网页版（React/TS/Vite）参考实现，**不在 .NET 解决方案内、未被 git 跟踪**。移植语义（遮罩 markProcess、分块 tileProc、下载 ensureModel）以它为基准；不要修改或提交它。
- 依赖只允许向下：`Inpaint.Core`（零包引用，图像布局转换）← `Inpaint.Inference`（仅 ONNX Runtime）← `Inpaint.App`（Avalonia + CommunityToolkit.Mvvm）。

## 关键模型语义（移植自网页版，改动前先对照 inpaint-web/src/utils.ts）

- **mask 张量：0 = 待修复，255 = 保留**。UI 白色笔触映射为 0：画布写入 `MaskLayer`（权威数据 = 每像素 1 字节灰度，255 = 待修复），推理经 `ImageProcessing.MaskGrayToChw`（255 → 0）转换；语义与旧版从 BGRA 位图按 OpenCV 灰度权重提纯白等价。
- **遮罩分两层（`Controls/MaskLayer`）**：权威 `Data`（byte[]，全分辨率）+ 显示 `Overlay`（WriteableBitmap，长边 ≤ 2048）。涂抹同步写两层；overlay 必须与原图分辨率解耦——WriteableBitmap 涂抹失效后整张重传 GPU（无增量更新），全分辨率遮罩在大图上等于每帧上传数百 MB。画布 `PaintDisc` 与 VM 推理（`MaskLayer.Data` 直转 CHW）都走它。
- MI-GAN（`migan_pipeline_v2.onnx`）：输入 image `[1,3,H,W]` uint8（RGB）+ mask `[1,1,H,W]` uint8，前后处理都在模型内完成。
- Real-ESRGAN（`realesrgan-x4.onnx`）：输入 float 0..1 RGB CHW；64×64 tile、四周外扩 6px 重叠、越界钳制到边缘像素，核心区 52×52，输出 ×4。
- 位图侧统一 Bgra8888 紧凑布局，模型侧 RGB CHW（平面式）；转换全部在 Core。
- **超大图性能约束**：像素级大块工作（`new Bitmap(stream)` 解码、`ExtractBgra`、CHW 前后处理、`CreateBitmap`）一律放后台（`Task.Run`），UI 线程只留属性赋值与缩略图绘制；修复/超分有像素上限 guard（VM `InpaintMaxPixels`/`UpscaleMaxPixels`，超限明确报错不 OOM）；超分输出超 1 亿像素（VM `UpscaleConfirmPixels`，按 ×4 后输出计）时先经 VM `ConfirmUpscaleAsync` 回调弹模态确认窗 `ConfirmWindow`（默认焦点「取消」，未接线按取消处理）再执行；历史裁剪除 `MaxHistory` 节点数外还有总字节预算（VM `HistoryByteBudget`，超大图自动收缩保留张数）；空闲悬停不触发画布整帧重绘（画笔光标环不可见时跳过 `InvalidateVisual`）。

## 导出压缩（Services/ImageExporter）

- 保存=导出：工具栏「导出…」/历史节点右键先弹模态 `ExportWindow`（格式 PNG/JPEG/WebP + 质量滑块 + **实时预估大小** + **1:1 取样预览**），确认后经 VM `ExportDialogProvider` 把 `ExportChoice`（含**编码好的字节**）带回，文件选择器按所选格式过滤扩展名。预估即编码——`ExportViewModel` 防抖 400ms 后整图编码一次，确认时参数命中缓存直接落盘不再重复编码；缓存是**单条目**（大图每份编码数 MB，不按质量档囤积），收益场景是估算在跑时切回上一组参数秒恢复，迟到的过期结果靠 generation 代数检查丢弃。
- **1:1 取样预览 + 全图导航器**：预估完成后把真实字节解码回位图（`PreviewResult`，先换引用再 Dispose 旧图，Detach 清空），预览=落盘内容（JPEG 白底合成也如实可见）。交互：上窗 1:1 取样——按住=切原图对比、拖动=微调取样中心（`CalculateCropTranslate` 钳制到图像边缘、图小于视口整体居中）；下窗全图导航器（原图渲染、打开即有内容）——高亮框标出取样区在整图的位置，按下即跳转、拖动跟随（`CalculateThumbLayout` 信箱布局不放大超过 1:1、`ThumbPointToImage` 坐标映射）。**四条 Avalonia 渲染坑（探针实测）**：① Image 控件自身会裁掉超出 Bounds 的绘制，必须设 Width/Height=位图像素尺寸让其铺满，再由 Border 的**显式 `Clip`**（RectangleGeometry）裁出取样窗；② `ClipToBounds` 不行——它的裁剪发生在子项自身坐标系、会跟着 RenderTransform 一起移动，负平移直接把可见区移没；③ 显式尺寸的子项在 Panel 里默认**居中**摆放（Stretch 对齐 + 显式宽高 → 居中），会叠加 ((槽宽-宽)/2, …) 的基底偏移把平移后的内容推出视口，须 Left/Top 对齐；④ `Stretch.Fill` + 显式像素尺寸强制 1 图像像素=1 DIP 的真 1:1，不受文件 DPI 元数据影响。
- 编码器用 Avalonia 自带的 Skia（`SKImage.Encode`，libpng/libjpeg-turbo/libwebp），**零新增原生依赖**；SkiaSharp 经 Avalonia 传递引用，勿再显式加包。**alpha 约定（探针实测）**：Avalonia 解码的位图缓冲是**预乘 alpha**（且 PNG 源解码为 Rgba8888，`ImageExporter.ExtractBgra` 统一转紧凑 BGRA 并交换红蓝），编码时按 `SKAlphaType.Premul` 声明；历史树新生成的位图全不透明，两者一致。JPEG 无 alpha：编码前把非不透明像素合成到白底（预乘公式 `c + 255 - a`，半透明红叠白底=粉色不是纯红），全不透明走零拷贝快路径；PNG/WebP 保留 alpha。质量参数 PNG 忽略（键里也不参与，来回调质量命中同一份缓存）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cholf5/inpaint](https://github.com/cholf5/inpaint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
