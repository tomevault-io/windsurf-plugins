---
trigger: always_on
description: 面向后续编码代理/维护者的项目速览与操作约定。当前目录为本仓库根目录。
---

# AGENTS.md

面向后续编码代理/维护者的项目速览与操作约定。当前目录为本仓库根目录。
最后同步：2026-07-29。

## 0. 总原则

- 本工程是 **Slay the Spire 2 Android 重构移植/启动器工程**，不是完整游戏源码仓库。
- 仓库只维护 Android shell、导入/版本管理逻辑、兼容包构建脚本、Android 兼容补丁源码与通用离线启动层源码；**不提交用户游戏 zip、解压后的完整游戏 payload、大型 Godot/Mono runtime、keystore**。
- `port-mod/` 是独立仓库 <https://github.com/ModinMobileSTS/sts2-android-compat> 的 **git submodule**。当前开发默认使用 flat matrix 模式：一个 checkout 读取 `port-mod/targets/active/*/target.json`，为多个目标版本构建 schema 2 family full compat 包；按游戏版本分支构建的 legacy 模式只作为显式回退/诊断路径保留。
- `offline-bootstrap/` 是独立边界的通用离线启动层目录：不读取 `port-mod/targets`、不接受 `ReferenceFlavor`、不静态引用 `sts2.dll`、不含 `STS2_TARGET_*` 分支；它输出 schema 2 `sts2-android-offline-bootstrap.zip`，仅在没有已安装 full compat 包按 payload SHA/version 命中时自动作为最低优先级 fallback。wildcard 不是未来版本兼容认证：运行时按 API 形状解析已知合约，未知语义 fail closed；probe v2 只有真实 ModelDb two-phase 完成后才标记 `ready`，已知终态失败的相同 pack id/target/compat version/source zip SHA/payload SHA tuple 不再自动匹配。
- 新增或修改功能时必须同步文档：用户可见/长期维护说明优先更新 `README.md` / `doc/`；变更流水/changelog 只写入 `.agent/agent-docs/changelog/`（不提交），因为它主要服务 agent 接力，不作为公开仓库文档。历史 `docs/` 已移到 `.agent/historical-backup/docs/` 本地备份，不再作为公开文档入口。`AGENTS.md` 是 agent/维护者专用操作约定；本地 agent 草稿、报告、worktree、参考 clone、历史备份与 agent 文档放入 `.agent/`，该目录不追踪。
- 完成用户要求的修改后，请用脚本构建一个 importer 版本 APK 便于测试：

```bash
tools/package/build_importer_apk.sh
```

- 寻找原版代码和其他关键参考内容时，请从全局配置里读取信息： .env 和 local.properties

## 1. 项目定位

Android 侧拆成三层维护：

1. **Android shell / launcher / 附加设置**
   - APK 默认进入 `GameSettingsActivity`，不是直接进入游戏。
   - 负责首次向导、本地 PC 游戏 zip 导入、Steam 登录/游戏下载、本地存档快照、Steam Cloud 与 WebDAV 云存档、私有目录管理、游戏版本/兼容包管理、启动 Godot Activity、日志/文件浏览、存档备份、MOD 管理。
2. **原版游戏 payload**
   - 用户本地提供 `SlayTheSpire2.zip`，或使用自己拥有 STS2 的 Steam 账号从 SteamPipe 下载。
   - 导入/下载后安装到 `<files>/payloads/<payload_id>/game/`；版本/配置切换只切换 launch profile 指针，不再复制完整 PCK/解压目录。
   - “版本”页可为同一个 payload 创建多个 `<files>/instances/<profile_id>/instance.json` 启动配置，并分别选择兼容包、存档/设置、MOD 使用全局目录或隔离目录；删除游戏本体或兼容包不会删除启动配置，启动时再提示缺失项。
   - 直装版构建时可临时内置 zip 到 APK assets，但构建脚本退出会清理，不能提交。
3. **Android 兼容包 / Harmony patcher**
   - `port-mod/STS2AndroidPortCompat` 编译输出 full compat `STS2Mobile.dll`。
   - `port-mod/overlay` 打包输出 full compat `port_compat.pck`。
   - legacy schema 1 兼容包 zip 形态为：`compat_manifest.json` + `STS2Mobile.dll` + `port_compat.pck` + `SHA256SUMS`。
   - flat schema 2 family 包 zip 形态为：`compat_manifest.json` + `variants/<target_id>/STS2Mobile.dll` + `variants/<target_id>/port_compat.pck` + `SHA256SUMS`；启动配置用 `compat_pack_id` + `compat_target_id` 指向具体 variant。
   - 兼容包不是普通用户 MOD；它由 launcher/Godot runtime 在游戏早期加载，用来 patch 原版 PC 程序集并让普通 MOD 系统在 Android 上工作。
4. **通用离线启动层 / offline bootstrap**
   - `offline-bootstrap/src/STS2OfflineBootstrap` 编译输出同名 `STS2Mobile.dll`，只为复用 patched runtime 入口 ABI；`ModelDbRuntimeContract` 按 API 形状解析无参 `ModelDb.Init()` 与默认 null `Init(Type[]?)`，显式注入集合保留原版路径，未知参数/返回语义拒绝接管。
   - `offline-bootstrap/overlay` 打包输出最小有效 `port_compat.pck`，默认不替换游戏资源。
   - schema 2 zip 形态为：`compat_manifest.json` + `variants/offline-any/STS2Mobile.dll` + `variants/offline-any/port_compat.pck` + `SHA256SUMS`，manifest 必须声明 `pack_kind=offline-bootstrap`、`match_mode=offline-wildcard`、`versions=["*"]`；当前 `compat_version=0.2.0-dev`、`probe_contract=offline-bootstrap-v2`。
   - Java 安装校验只允许这种受限 offline 包使用 `*`；普通 schema 1/full compat 包出现版本或 SHA 通配符应拒绝安装。probe v2 的终态失败会阻止同一 tuple 再次自动推荐，但用户手工绑定的 profile 仍可在失败详情对话框中显式重试。

## 2. 当前支持版本矩阵

`port-mod` 当前默认跟踪 `main`，并采用 flat matrix 打包模式：不按游戏版本切开发分支，而是从当前 checkout 的 `targets/active/*/target.json` 循环编译多个 target，并输出一个 schema 2 family 包。`compat/*` 分支只作为 legacy 发布包对照、回退诊断或历史维护入口；legacy 打包模式会通过临时 worktree 同时构建多个 schema 1 内置兼容包，仅在显式设置 `COMPAT_PACK_BUILD_MODE=legacy` 时使用。

| 通道 | 游戏版本 | Steam 分支 | 原版/解包引用配置 | legacy submodule 分支 | compile gate `ReferenceFlavor` | flat target id | legacy 兼容包 id |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 正式/稳定 | `v0.103.2` / `v0.103.3` | `public` | `.env`: `STS2_ORIGINAL_V103_REFERENCE_DIR` 或 `STS2_ORIGINAL_V103_ROOT` | `compat/v0.103.2` | `original` | `v0.103.x` | `sts2-android-compat-v0.103.x` |
| Beta 旧测试 | `v0.106.1` | `public-beta` | `.env`: `STS2_ORIGINAL_V1061_REFERENCE_DIR` 或 `STS2_ORIGINAL_V1061_ROOT` | `compat/v0.106.1-beta` | `original-v0.106.1` | `v0.106.1-beta` | `sts2-android-compat-v0.106.1-beta` |
| Beta 旧测试 | `v0.107.0` | `public-beta` | `.env`: `STS2_ORIGINAL_V1070_REFERENCE_DIR` 或 `STS2_ORIGINAL_V1070_ROOT` | `compat/v0.107.0-beta` | `original-v0.107.0` | `v0.107.0-beta` | `sts2-android-compat-v0.107.0-beta` |
| 正式/稳定 | `v0.107.1` | `public` | `.env`: `STS2_ORIGINAL_V1071_REFERENCE_DIR` 或 `STS2_ORIGINAL_V1071_ROOT` | — | `original-v0.107.1` | `v0.107.1` | — |
| 正式/稳定 | `v0.108.0` | `public-beta` | `.env`: `STS2_ORIGINAL_V1080_REFERENCE_DIR` 或 `STS2_ORIGINAL_V1080_ROOT` | — | `original-v0.108.0` | `v0.108.0` | — |
| Beta 当前测试 | `v0.109.0` / `v0.109.1` | `public-beta` | `.env`: `STS2_ORIGINAL_V1090_REFERENCE_DIR` 或 `STS2_ORIGINAL_V1090_ROOT`（历史变量名，当前指向最新 v0.109.1 引用） | — | `original-v0.109.0` | `v0.109.0`（稳定 id，显示为 v0.109.x） | — |

关键文件：

- `.gitmodules`：`port-mod` submodule GitHub URL 与默认 branch（`main`）。
- `tools/android/bundled-compat-packs.json`：legacy 内置兼容包列表，当前包含 `compat/v0.103.2`、`compat/v0.106.1-beta` 与 `compat/v0.107.0-beta`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ModinMobileSTS/Sts2MobileLauncher](https://github.com/ModinMobileSTS/Sts2MobileLauncher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
