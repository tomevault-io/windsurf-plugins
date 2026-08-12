---
trigger: always_on
description: - `ExpressPackingMonitoring.sln` is the main solution.
---

# Repository Guidelines

## Project Structure & Module Organization

- `ExpressPackingMonitoring.sln` is the main solution.
- `ExpressPackingMonitoring/` contains the WPF application, including XAML views, view models, services, SQLite access, recording logic, and `Web/index.html`.
- `ExpressPackingMonitoring.Launcher/` contains the small launcher executable used by the clean package layout.
- `Tools/Publish-CleanPackage.ps1` creates the per-user Setup, distributable directory, LZMA2 solid 7z, compatibility zip, update manifest, launcher manifest, optional AppPatch, and a separate LauncherPatch.
- `Scripts/快递助手订单推送.user.js` is the browser userscript for order push integration.
- `Image/` stores README and project screenshots. `Test/HTML/` contains captured sample pages for script/debug reference, not an automated test suite.

## Build, Test, and Development Commands

```powershell
dotnet restore ExpressPackingMonitoring.sln
dotnet build ExpressPackingMonitoring.sln -c Debug
dotnet run --project ExpressPackingMonitoring
pwsh -NoProfile -File Tools\Publish-CleanPackage.ps1
pwsh -NoProfile -File Tools\Test-Release-Automated.ps1
```

- `restore` downloads NuGet dependencies.
- `build` verifies the WPF app and launcher compile.
- `run` starts the main app locally.
- `Tools\Publish-CleanPackage.ps1` produces the clean release layout with the root launcher and `app\` payload.
- `Tools\Test-Release-Automated.ps1` runs the isolated WPF smoke test, userscript concurrency/routing tests, and headless Web UI acceptance suite.

## Runtime and Distribution Notes

- The publish script generates a directory package and a matching `.zip`.
- The clean package root should mainly contain `ExpressPackingMonitoring.exe` and `app\`; the real app payload, dependencies, Web files, LibVLC files, and `tools\ffmpeg.exe` live under `app\`.
- Release packages must not include `config.json`, `videos.db`, cache files, logs, recordings, or other local runtime data.
- Runtime data is stored under `%LOCALAPPDATA%\ExpressPackingMonitoring\`, so normal upgrades keep user configuration and database records.
- `ffmpeg.exe` may be resolved from `app\tools\ffmpeg.exe`, the application runtime directory, or the system `PATH`.
- 正式发布基线固定为 FFmpeg 4.4.1 Essentials（兼容 Win7 老显卡 NVENC API 11.1）。AV1 硬件编码不作为产品能力，选择 AV1 时会自动回退 H.265；双 FFmpeg 基线方案（8.0.1 + 4.4.1）已评估但暂不实施。高级用户可在 Win8+ 自行替换 `app\tools\ffmpeg.exe` 获取新能力，官方不保证支持。
- FFmpeg CLI 选项存在版本差异，禁止假设某个选项在所有版本可用：FFmpeg 8.x 已移除 RTSP 的 `-stimeout`，4.4.x 的 `-timeout` 在 RTSP 上会挂起，因此网络摄像头解码参数不传 socket 超时选项，由应用层 15 秒连接超时与断流看门狗兜底；`-fps_mode` 仅 5.1+ 可用，旧版本必须回退 `-vsync passthrough`。参数兼容策略集中在 `NetworkCameraSource.BuildArguments` 一处维护。
- 修改任何 ffmpeg 调用（录制编码、网络摄像头解码、编码器探测、音频/TTS 探测）前，必须用发布基线 ffmpeg（`Tools/ffmpeg-baseline.json` 锁定的 4.4.1）和至少一个其他受支持主版本（如 8.0.1）实际验证；只在本机某个 ffmpeg 上通过不算验证完成。
- AppPatch 不携带 `ffmpeg.exe`，用户机器可能长期保留旧完整包的不同 ffmpeg 版本；应用层逻辑必须对版本差异保持兼容，不能依赖 AppPatch 更新 ffmpeg。
- LibVLC 随包收录全部播放相关插件（解码/解封装/字幕/滤镜/输出），仅排除与本地录像回放无关的目录（access_output/mux/services_discovery/stream_out/visualization/lua）；发布时移除设计时程序集（ReachFramework、WinForms Design）。收录与排除规则集中在 `ExpressPackingMonitoring.csproj`，新增播放能力需要插件时按同目录模式追加。
- `Scripts/快递助手订单推送.user.js` is the browser userscript used for order push integration.
- Edge TTS is the default online voice path. Kokoro local TTS models and runtime dependencies are optional and should not be bundled unless explicitly intended.
- Full packages include the generated default Edge TTS cache. AppPatch packages must exclude TTS cache files.

## Update & Release Workflow

- Users should start the root launcher. The launcher starts the app immediately, checks updates in the background, downloads verified AppPatch packages into `%LOCALAPPDATA%\ExpressPackingMonitoring\cache\updates`, and installs pending patches on the next launcher run.
- The main app may update the root launcher through the optional, separately verified `launcher_package` descriptor. It must wait for the old launcher process to exit, use the shared update mutex, replace only the standard root launcher, verify the result, and restore the previous launcher on failure.
- AppPatch packages are fixed-baseline cumulative patches. The AppPatch baseline is specified by `-PatchBaselineVersion` and defaults to `0.0.18`, but scripts may allow overriding it when a new formal baseline is chosen. It is independent from the launcher baseline.
- Keep update URLs configurable through environment variables or `.env`. The default update check URL is GitHub releases latest API; `.env` may point to another release provider.
- 主程序默认版本号维护在 `ExpressPackingMonitoring/ExpressPackingMonitoring.csproj` 的 `<Version>`：每次发布前手动更新为本次版本，并与 `vX.Y.Z` 标签一致；`Publish-CleanPackage.ps1` 仍通过 `-p:InformationalVersion` 注入发布版本，本地与 CI 普通构建显示该默认版本。
- 发布打包命令（每次发布按此执行）：
  - `pwsh -NoProfile -File Tools\Publish-CleanPackage.ps1 -Version <X.Y.Z> -BaselineAppDir "package\PackingProof+v<上一正式版>\PackingProof+v<上一正式版>\app"`
  - `-BaselineAppDir` 必须指向上一个完整包的 `app` 子目录（内含 `tools\ffmpeg.exe`）；指到包根目录会导致 AppPatch 不生成
  - `-ReuseExistingLauncherBaseline` 仅当本次发布标签与锁定启动器基线标签相同（同版本重发）时使用；普通新版本不要传该参数，脚本会自动复用锁定基线启动器且不生成 LauncherPatch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PackingProof/PackingProof-Desktop](https://github.com/PackingProof/PackingProof-Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
