---
trigger: always_on
description: 本文件是 `windows/` 目录下的权威指南，面向在此工作的编码 agent（Claude Code、Codex 等）和人。
---

# AGENTS.md —— Ncrust for Windows

本文件是 `windows/` 目录下的权威指南，面向在此工作的编码 agent（Claude Code、Codex 等）和人。
仓库级约定（提交规范、「一个逻辑单元一个 commit」等）见根目录 `AGENTS.md`；根文件里
Android 专属的章节不适用于这里。

**状态**（2026-09-27）：**M1 进行中，已可日常点播。** 包版本 0.1.2.0。

- `Ncrust.Core`：协议、加密、全部端点（含相似歌曲）、队列状态机、INFINITY 续播取数（`InfinityFeeder`）、
  音质阶梯、缓存与持久化、云收藏库、10 段均衡器（DSP + 预设），Core 测试 230 个通过。
- `Ncrust.App`：标准汉堡菜单外壳（`NavigationView`）+ 自定义标题栏、独立登录层（WebView2 + 二维码）、
  `PlaybackEngine`（滑动窗口 / MediaBinder / 降级 / 上报 / 均衡器 / INFINITY 与私人 FM 续播）、
  Groove 式传输栏与 Composition 播放器层（卡片右栏：歌词 / 播放队列）、首页（含私人 FM 卡）、搜索页、
  专辑 / 歌手 / 歌单详情页、音乐库页、设置页与 10 段均衡器子页面；歌曲与专辑 / 歌单磁贴的右键菜单。
- `Ncrust.Audio`：均衡器音效组件（`IBasicAudioEffect`，挂在 `MediaPlayer` 上）。
- `Kanesumi.Xaml`：token、缓动、按钮 / 列表 / 进度环样式、NavigationView 资源覆盖、`KanesumiAccent`（强调色跟随系统）、
  `MetroLyricsPanel`。

与 Android 主干功能已基本对齐（见「Android → Windows 映射」）；验收记录见 `docs/PLAN.md`。
下一步：搜索历史入口、窄窗口的歌词 / 队列、多选批量操作、8 种语言。

本文描述的是**已定的架构**，除「目录结构」里列出的现有文件外，其余都是待实现的设计。
写代码时如果发现与本文冲突，先改本文、再改代码，并在 commit 里说明原因。

相关文档：

- `windows/docs/KANESUMI_XAML.md` —— 控件迁移规格（从 Kanesumi-sec-a 移植）
- `spec/README.md` —— 跨平台规格与夹具约定
- `spec/design/tokens.json` —— 设计 token（颜色、字号、缓动、时长、尺寸）

## 立项决策

| 决策 | 结论 | 原因 |
|---|---|---|
| 仓库 | monorepo：`windows/` 与 `spec/` 放在 Ncrust 仓库里，Android 的 `app/` 不挪 | 改协议时，spec 与两端实现可以在同一个 commit 里改完；零迁移成本 |
| 目录名 | `windows/`（按平台命名） | 与 `app/`（Android）并列，语义清楚 |
| UI 框架 | **UWP + WinUI 2**（`Microsoft.UI.Xaml` 2.8.x） | arc-deck 已验证：平台免费提供文本、IME、滚动、虚拟化、无障碍，框架搭起来就成型 |
| WinUI 3 / Windows App SDK | **短期内不考虑。不要引入，不要主动提议迁移** | 项目负责人的决定 |
| 运行时 | .NET Native（`UseDotNetNativeToolchain`），C# `LangVersion` 10 | arc-deck 已在本机验证。「UWP on 现代 .NET」只在 M0 花半天评估，结论记入本文，不切换主线 |
| 视觉 | 控件从 Kanesumi-sec-a 迁移为 **Kanesumi.Xaml**，不用 WinUI 默认的 Fluent 外观；Windows 允许在 Kanesumi 语言之上做**新设计** | 与 Android 在语言 / 控件层同源，但**不逐像素对齐**；桌面端**以美观为先**，可另设布局与视觉 |
| 底色 | 深色 `#000000`，与 Ncrust Android 一致 | 同一个产品两端一致；不跟 Ether 桌面扇区的 `#1A1A1A` |
| 代码共享 | 不共享实现，共享 `spec/` 夹具 | 见 `spec/README.md` |
| 业务核心 | `Ncrust.Core` 用 netstandard2.0，**不依赖 WinRT，也不依赖 UI** | 测试可以用普通 `dotnet test` 秒级跑完；以后换运行时也能原样复用 |
| 播放 | `MediaPlayer` + `MediaPlaybackList` + `MediaBinder` | 系统提供无缝播放、SMTC、后台音频；延迟绑定正好解决 URL 过期问题 |
| 动画 | `Windows.UI.Composition`，单一 progress 标量驱动 | 对应 Android 的「单一 progress + `graphicsLayer`」，在合成线程执行 |
| 平台控件优先 | **能用 WinUI / 平台原生控件的就用原生**（NavigationView、Pivot、Slider、ContentDialog…），只做资源键级的 Kanesumi 覆盖；不再自绘替代品 | 负责人实测自绘 Tab 行体验差（2026-09-26）。原生控件自带键盘、UIA、触屏手势与系统一致的交互 |
| 强调色 | **跟随 Windows 系统强调色**（设置 → 个性化 → 颜色，Windows 10 / 11 都有）；读不到时回落内置云杉 `#1DB954`。平台控件直接用 SystemAccentColor，Kanesumi 的 `KPrimaryBrush` 由 `KanesumiAccent.FollowSystem` 同步并实时跟随；`KOnPrimaryBrush` 按亮度选黑 / 白 | 负责人要求。应用图标的绿色是品牌色，不随强调色变 |
| 外壳导航 | **标准汉堡菜单**：WinUI 2 `NavigationView`（自适应展开 / 紧凑 / 最小），不用自绘 ListView 侧栏或窄窗底部导航 | 负责人要求标准汉堡菜单；参考 Groove Music。平台控件自带自适应、返回按钮、键盘与 UIA |
| WinUI 2 样式版本 | `XamlControlsResources ControlsResourcesVersion="Version1"` | Windows 10 / Groove 一代的直角样式；Version2 是 Windows 11 圆角 + 中灰圆角内容面板，与 Kanesumi 冲突 |
| 按钮 | **微软原生样式**：主操作 `AccentButtonStyle`，次要操作平台默认 `Button`，图标按钮 `AppBarButton`（`LabelPosition=Collapsed`，同 Groove 播放栏），播放卡片的播放键是强调色按钮。Kanesumi 的 `Metro*ButtonStyle` 不再使用 | 负责人（2026-09-27）：按钮不用 Ncrust Android 的设计，用微软的 |
| 磁贴间距 | 桌面横向 20 / 纵向 28、磁贴 176（`MetroGridTileStyle`），**有意偏离** tokens 的 `gridSpacing = 2` | 2 的拼贴缝在手机上是 Kanesumi 的拼贴感，桌面一整面墙显得太紧张（负责人反馈，2026-09-27） |
| 图标 | 界面图标用 **Segoe MDL2 Assets**（显式指定 `FontFamily`）；应用图标是整块绿底唱片纹 | Groove 同源的原生图标字体；不打包 Material Icons（原 KANESUMI_XAML 的设想已撤回） |
| DI / MVVM 框架 | 不用。与 Android 一样用单例充当服务定位器；`INotifyPropertyChanged` 手写；只用 `x:Bind` | 依赖越少，.NET Native 的反射问题越少；`x:Bind` 是编译期绑定 |

## 构建、测试与运行

环境：VS 2022 Build Tools（已验证 MSBuild 17.14）+ UWP 工作负载（Windows SDK 10.0.22621）、
.NET 9 SDK。

UWP 项目**只能用 MSBuild**（不支持 `dotnet build`），并且**用 Release 配置**：
Debug 版的 .NET Native 依赖一个默认不预装的调试运行时。解决方案里也只有 `Release|x64` 一种配置。

```powershell
$msbuild = "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe"

& $msbuild windows\Ncrust.Windows.sln /t:Restore /p:Configuration=Release /p:Platform=x64
& $msbuild windows\Ncrust.Windows.sln /p:Configuration=Release /p:Platform=x64
# 产物：windows\src\Ncrust.App\AppPackages\Ncrust.App_<版本>_x64_Test\Ncrust.App_<版本>_x64.msix

# Core 单元测试（net9.0，不需要部署 UWP；同时检查 spec/design/tokens.json）
dotnet test windows\tests\Ncrust.Core.Tests
```

本地运行（需要开启 Windows「开发者模式」）：注册 .NET Native 编译后的布局，再启动。

```powershell
Add-AppxPackage -Register windows\src\Ncrust.App\bin\Release\ilc\AppxManifest.xml
explorer.exe "shell:AppsFolder\TakahashiRinta.Ncrust_98kk3q0vty278!App"

# 未处理异常写在这里：
Get-Content "$env:LOCALAPPDATA\Packages\TakahashiRinta.Ncrust_98kk3q0vty278\LocalState\crash.log"

# 卸载（注册指向构建输出目录，清理 bin 之前先卸载）：
Get-AppxPackage TakahashiRinta.Ncrust | Remove-AppxPackage
```

**改了 `Package.appxmanifest`（包括构建自动写入的音效类注册）就要提高版本号**，否则重新注册报
`0x80073CFB`；卸载重装可以绕过，但会清掉 LocalSettings（登录态、音量、均衡器设置都没了）。
注册后第一次启动偶尔没反应，再启动一次即可。

界面验证工具：M0 时从 arc-deck `uwp/tools/shot/` 移植到 `windows/tools/shot/`
（`ShotWindow`、`Uia`、`Verify`、`Contrast`、`ResourceAudit` 等）。移植时注意「已知的坑」里
关于截图脚本的两条。

## 目录结构

```
windows/
├── AGENTS.md
├── Ncrust.Windows.sln          # 手写维护（dotnet sln 无法添加 UWP 项目），仅 Release|x64
├── docs/
│   ├── KANESUMI_XAML.md
│   └── PLAN.md                 # 施工单与设备验收记录
├── src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuitaristRin/Ncrust](https://github.com/GuitaristRin/Ncrust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
