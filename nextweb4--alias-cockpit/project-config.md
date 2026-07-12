---
trigger: always_on
description: - 当前目录 `D:\Codex\BM` 已初始化为 WinUI/.NET 本地应用项目：根目录包含 `AliasCockpit.slnx`、`Directory.Build.props`、`Directory.Packages.props`、`.editorconfig`、`AGENTS.md`。
---

# AGENTS.md

## 1. 项目结构

- 当前目录 `D:\Codex\BM` 已初始化为 WinUI/.NET 本地应用项目：根目录包含 `AliasCockpit.slnx`、`Directory.Build.props`、`Directory.Packages.props`、`.editorconfig`、`AGENTS.md`。
- `src/AliasCockpit.App/` 是 Windows WinUI 3 本地应用壳，只负责窗口、页面、ViewModel、剪贴板和桌面交互；当前主页面是本地 Email Alias Expander 工具，并通过 Core 仓储接口保存输入邮箱历史和 alias 标记。
- `src/AliasCockpit.App/Assets/AppIcon.ico` 是应用图标源；必须同时通过 App 项目的 `ApplicationIcon`、`MainWindow.AppWindow.SetIcon` 和 MSI 快捷方式 `Icon` 写入，不得只替换 PNG/ICO 文件后直接打包。
- `src/AliasCockpit.Core/` 是纯业务核心，当前包含 Alias 生成算法、Gmail/Outlook 邮箱别名裂变、保存输入邮箱接口、熵估算、审计事件模型、Provider 能力模型、ProviderAccount 和 Provider adapter 抽象，不得引用 WinUI、HTTP、SQLite 或系统剪贴板。
- `src/AliasCockpit.Infrastructure/` 是基础设施层，当前包含 SQLite alias/saved email/provider account/audit log 仓储、Windows Credential Manager secret store、SimpleLogin/addy.io mock adapter 和 SimpleLogin/addy.io HTTP adapter；不得反向引用 App/UI。
- `tests/AliasCockpit.Core.Tests/` 是核心单元/压力测试。
- `tests/AliasCockpit.App.Tests/` 是 App/ViewModel 单元测试，不启动 WinUI 窗口；当前覆盖有标记/未标记筛选行为。
- `tests/AliasCockpit.Infrastructure.Tests/` 是 SQLite 集成测试、Windows Credential Manager 集成测试和 mock Provider adapter 测试。
- `benchmarks/AliasCockpit.Benchmarks/` 是无第三方依赖的基础性能测量入口。
- `docs/research/` 保存调研证据，`docs/architecture/` 保存方案选型与产品架构，`docs/security/` 保存安全威胁模型。
- `.tools/` 用于本地隔离工具链和下载缓存，已通过 `.gitignore` 忽略，不属于产品源码。
- 新增代码前必须先明确应用边界：Windows 本地应用、邮箱别名生成/管理/同步/导入导出，不得把主产品实现成网页应用。
- 后续新增目录时必须保持职责清晰：应用 UI、核心业务、数据持久化、同步、Provider/API 集成、安全/加密、测试与文档应分层放置。
- 调研、选型、安全、架构文档必须继续放在 `docs/` 对应子目录，不得散落在根目录。
- GitHub 发布审计记录位于 `docs/architecture/github-publishing-audit-2026-07-07.md`；Release 说明放在 `docs/release/`，不要把 release 说明散落到根目录。

## 2. 运行命令

- 本地 SDK 验证：`.\.tools\dotnet\dotnet.exe --info`。
- 本地 runtime 验证：`.\.tools\dotnet\dotnet.exe --list-runtimes` 必须能看到 `Microsoft.NETCore.App 8.0.28` 和 `10.0.9`；当前 `.slnx` 由 .NET 10 SDK 编排，产品项目目标框架为 .NET 8。
- 桌面应用运行候选命令：`.\.tools\dotnet\dotnet.exe run --project src\AliasCockpit.App\AliasCockpit.App.csproj`。
- 桌面应用当前主页面做本地 Gmail/Outlook 邮箱别名裂变，并读写 `%LocalAppData%\AliasCockpit\aliases.sqlite` 中的保存输入邮箱历史与 alias 标记；启动时不得调用 Provider HTTP adapter，不得联网。
- SQLite 开发库 `%LocalAppData%\AliasCockpit\aliases.sqlite` 尚未加密，只能保存 alias 元数据、颜色、使用位置、用途、保存的输入邮箱地址和未来 `secret_ref`，不得存放 token/secret。
- Provider token / API secret 必须通过 `WindowsCredentialManagerSecretStore` 或后续等价 secret store 存储，SQLite 只能保存 `secret_ref`。
- Provider account metadata 通过 `SqliteProviderAccountRepository` 保存；SimpleLogin/addy.io HTTP adapter 支持 API key 校验、random/custom alias 创建、disable 和 delete，App 不得默认联网调用它们。
- 当前自动门禁验证 build/test/benchmark/format/publish/prune/zip/MSI/setup EXE/process smoke/basic UI smoke；完整 UI 自动化仍需后续补充。
- 完整发布验证脚本：`powershell -NoProfile -ExecutionPolicy Bypass -File scripts\verify-release.ps1`；该脚本会执行 build/test/benchmark/format/publish、裁剪当前未使用的 WinAppSDK AI/WebView/Widgets/诊断文件、重建 portable zip、生成 MSI、生成 setup EXE 安装器、验证 MSI 数据库、抽取 setup EXE 确认内嵌 MSI、检查 zip 根目录 exe，对 publish/artifact 两处 exe 做 5 秒启动冒烟，并通过 UI 输入 Gmail 地址验证复制出的生成别名。
- 当前可发布 x64 exe：`.\.tools\dotnet\dotnet.exe publish src\AliasCockpit.App\AliasCockpit.App.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=false -v minimal`，输出位于 `src\AliasCockpit.App\bin\Release\net8.0-windows10.0.26100.0\win-x64\publish\AliasCockpit.App.exe`。
- 发布后必须至少做一次启动冒烟验证：从 `publish` 目录启动 `AliasCockpit.App.exe`，确认进程不会立即退出；当前 Windows App SDK 2.2.0 + .NET 8 self-contained 组合已通过该验证。
- 当前便携交付包位于 `artifacts\AliasCockpit-win-x64-portable.zip`；它必须包含完整 publish 目录内容，不能只包含单个 exe。更新该包后必须检查 zip 根目录包含 `AliasCockpit.App.exe` 并从解压/复制后的 artifact 目录做启动冒烟验证。
- 当前 MSI 交付包位于 `artifacts\AliasCockpit-win-x64.msi`；它必须由 `scripts\package-msi.ps1` 从完整 publish 目录生成，不能只打包单个 exe。
- 当前 EXE 安装包位于 `artifacts\AliasCockpit-win-x64-setup.exe`；它必须由 `scripts\package-setup-exe.ps1` 生成，是内嵌 MSI 的 WiX Burn 安装器，不得把 `publish\AliasCockpit.App.exe`、快捷方式或 zip 内 app exe 当成“exe 安装包”交付。
- MSI 和 setup EXE 的安装入口必须显示 `Assets\AppIcon.ico`；`package-msi.ps1` 负责 Start Menu shortcut / ARPPRODUCTICON，`package-setup-exe.ps1` 负责 Burn `IconSourceFile`。
- GitHub Release 附件必须上传 `artifacts\AliasCockpit-win-x64-setup.exe`、`artifacts\AliasCockpit-win-x64.msi` 和 `artifacts\AliasCockpit-win-x64-portable.zip`；这些构建产物不得提交进 Git 历史。
- GitHub 发布脚本为 `scripts\publish-github-release.ps1`；它必须通过 `GITHUB_TOKEN`、`GH_TOKEN`、Git Credential Manager 或 Codex GitHub 集成 token helper 临时取 token，不得把 token 写入文件、Git remote 或日志。
- 发布目录瘦身使用 `powershell -NoProfile -ExecutionPolicy Bypass -File scripts\prune-publish.ps1 -PublishDir <publish-dir>`；该脚本只允许删除已审计的未用 WinAppSDK AI/WebView/Widgets/Workloads/诊断文件，执行后必须重新通过 publish 启动冒烟和 UI smoke。
- 不得删除 `Microsoft.InteractiveExperiences.Projection.dll`；实测删除后 WinUI 发布程序启动失败。
- MSI 构建工具固定为 WiX CLI `5.0.2`，通过 `.\.tools\dotnet\dotnet.exe tool install wix --version 5.0.2 --tool-path .tools\wix` 恢复到 `.tools\wix`；它是构建工具，不得作为产品运行依赖加入 App/Core/Infrastructure。
- setup EXE 使用 WiX BAL 扩展 `WixToolset.Bal.wixext` `5.0.2`；该扩展只用于构建安装器，不得作为产品运行依赖。
- MSI 生成后必须能通过 `.\.tools\wix\wix.exe msi validate artifacts\AliasCockpit-win-x64.msi`；`scripts\package-msi.ps1` 会把 Windows App SDK 本地化资源导致的 `File.Language` 表项归一化为 `0`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/alias-cockpit](https://github.com/NextWeb4/alias-cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
