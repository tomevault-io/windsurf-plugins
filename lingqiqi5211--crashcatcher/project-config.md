---
trigger: always_on
description: 记录应用崩溃、ANR 与原生崩溃的 root 模块。三块：`crates/` 是 root 守护进程与其库（Rust workspace，edition 2024）、`bridge/` 是 `app_process` 起的特权 Java 桥、`apps/manager` 是 Compose 管理器（AGP 9 内置 Kotlin，UI 走 MeowUI 复合构建）。`apps/crashdemo` 是故意崩溃的测试 app。
---

# CrashCatcher

记录应用崩溃、ANR 与原生崩溃的 root 模块。三块：`crates/` 是 root 守护进程与其库（Rust workspace，edition 2024）、`bridge/` 是 `app_process` 起的特权 Java 桥、`apps/manager` 是 Compose 管理器（AGP 9 内置 Kotlin，UI 走 MeowUI 复合构建）。`apps/crashdemo` 是故意崩溃的测试 app。

本文件是 agent 指南的入口，`CLAUDE.md` 只有一行 `@AGENTS.md`——Claude Code 只自动读 `CLAUDE.md`。

**对照物是 [AppErrorsTracking](https://github.com/KitsunePie/AppErrorsTracking)**：它从 system_server 里 Hook ActivityManager，所以每个 Android 大版本都要适配，且依赖 Zygisk + Xposed。本项目**零注入**——不进任何进程，只读平台已经对 root 公开的那几个口子。看到「要不要 Hook 一下」的念头时先回到这条。

## 工作规程

- **改完先编译再提交**，用户明确要求过。Kotlin 侧 `apps/manager` 跑 `./gradlew :app:assembleDebug`（分钟级，release 走 R8 更久）；Rust 侧 `.\tools\cargo.ps1 clippy --workspace --all-targets` + `test --workspace`。
- **Windows 上 cargo 必须经 `tools\cargo.ps1`**：MSVC Build Tools 不在 PATH，裸 `cargo test` 链接失败（Git Bash 下 `link` 还会解析成 coreutils 的 `link`）。Android 交叉编译走 NDK clang，不受影响。
- **clippy 在本仓禁 `unwrap`/`expect`/`panic`**（见 [clippy.toml](clippy.toml)），CI 是 `-D warnings`。守护进程以 root 跑在别人手机上，一个 panic 就是崩溃记录静默停摆。
- **工具链钉在 [rust-toolchain.toml](rust-toolchain.toml)**：edition 2024 下 rustfmt 的 import 排序会随版本漂移，不钉就会出现「本地格式化过、CI 仍报 fmt 失败」。要升先改这里，再 `cargo fmt --all` + clippy 一起提交。
- submodule 是整个 UI 层（MeowUI，内含 miuix），首次 clone 后 `git submodule update --init --recursive`。
- 保留用户未提交的改动；不用破坏性 reset/checkout；不改也不输出 `local.properties` 与 `keystore.properties`。
- **提交信息不带协作者标识**，作者 `柒柒喵 <lingqiqi233@gmail.com>`。`<scope>: <summary>`，scope 取 `daemon`/`bridge`/`module`/`manager`/`demo`/`build`/`ci`/`docs`；主题行 ≤ 72 字符、无句尾句号；**body 只讲代码里看不出的根因与取舍**，不逐文件复述 diff。
- `docs/` 与 `dist/` 不入库（前者是本地设计稿，后者是产物且带签名 pin）。

## 技术栈

Rust edition 2024（rusqlite bundled、zstd、serde_json、tracing）；Kotlin + Compose + kotlinx.serialization + MeowUI；桥是纯 Java 对着 `android.jar` 编译再 `d8` 成 dex。

**版本与坐标唯一真源**：Rust 侧 [Cargo.toml](Cargo.toml) 的 workspace 表，Kotlin 侧 `apps/manager/gradle/libs.versions.toml`，**发布版本号在 [version.properties](version.properties)**——Gradle 读它当 versionName（versionCode 由 `major*10000+minor*100+patch` 推导），`cch-packager` 读它写 `module.prop`。**文档不复述版本号。**

## 代码地图

```
crates/
├── cch_daemond/   服务本体：core（请求分发）server transport collectors packages bridge_broker
├── cch_logd/      logd 二进制协议：event（am_crash/am_anr）+ crash（FATAL EXCEPTION）
├── cch_dropbox/   DropBox 条目；cch_tombstone/ protobuf tombstone；cch_anrfile/ ANR 转储
├── cch_watcher/   inotify；cch_merge/ 多源同一崩溃合一；cch_model/ 领域模型 + RecordId
├── cch_store/     SQLite 索引 + zstd 正文文件（read/write/retention/payload/schema/sql）
├── cch_wire/      JSON 协议（rpc/dto/event/frame/bridge）+ tests/vectors.rs
├── cch_auth/      按签名证书 pin 鉴权；cch_apk_sig/ APK v2/v3 签名块解析
├── cch_config/    配置文档与 patch；cch_settings/ Android settings（接管系统弹窗）
└── cch_packager/  发布工具：三 ABI 守护进程 + 桥 dex + 模块 zip（子命令 module/manager-apk/bridge）
bridge/src/.../CrashCatcherBridge.java   通知、PackageManager 富化、跨用户启动 Activity
module/            service.sh（选 ABI、退避重启、把状态写进 module.prop 描述）+ customize.sh + service.d/
apps/manager/app/src/main/kotlin/.../crashcatcher/
├── data/daemon/   DaemonClient + WireRpc/WireModel/WireJson + DaemonRepositories
├── data/device/   readDeviceInfo（Build.* 快照）
├── domain/        model（LoadState/DomainError/DeviceInfo）+ repository（接口）
└── ui/            shell（CrashCatcherApp/Chrome/Destination/AppContainer）home crashes apps
                   settings detail components util theme
```

屏幕与 ViewModel 一一对应、`components/` 内各件职责都能从文件名读出，此处不复述。

## 架构

```
logd / DropBox / tombstone / ANR 文件
  → 各 collector → cch_merge 合一 → cch_store（SQLite 索引 + zstd 正文）
       ↑ root 守护进程 catcherd（module/service.sh 拉起，退避重启）
       ├→ 抽象 unix socket @crash_catcher_daemon_manager ──→ 管理器（按签名 pin 鉴权）
       └→ 抽象 unix socket @crash_catcher_daemon_bridge ──→ 特权 Java 桥（app_process）
                                                              通知 / 包名标签 / 启动 Activity
```

四字节大端长度前缀 + JSON 一帧；正文不进帧，走 memfd 经 `SCM_RIGHTS` 传描述符，太大或失败才退回分块读。

## 关键约束

### 采集与存储

**`RecordId` 必须从库里已有的最大 id 续号**（`Store::open` → `resume_ids()` → `RecordIdGenerator::resume_after`）。id 的高 48 位是**崩溃自己的**时间戳而非当前时间，所以重启后从零开始的生成器会对同一条 tombstone 再发一次同样的 id，插入撞主键，**崩溃被静默丢掉**，唯一痕迹是日志里一行 `UNIQUE constraint failed: crash_record.id`。

**`packages.xml` 从 Android 12 起是 ABX（二进制 XML）**，`read_to_string` 直接 "stream did not contain valid UTF-8"，守护进程会起不来并被 service.sh 反复重启。包信息一律走 `cmd package list packages -f`。

**守护进程由 service.sh 拉起，比 system_server 早**——那时 `cmd package` 全部失败，索引只有 uid、没有 APK 路径也没有系统标记，**每个应用都会被当成第三方**。所以「是不是系统应用」直接问 `cmd package list packages -s`（那就是 `FLAG_SYSTEM`，而不是猜分区前缀——`/system_ext` 正是漏掉后让「记录系统应用」看起来失灵的那个），并在启动后重试补全（`complete_package_index`）。鉴权失败触发的重载可能又落在 PM 不可用的时刻，所以替换索引要经 `install_packages`，让新索引继承已知的系统标记，否则分类会退回开机早期的状态直到下次重启。

**存储 schema 只升不降**：库升到 v2 后，旧版本守护进程读到 `user_version` 更大会返回 `SchemaTooNew` 并拒绝启动，表现为 service.sh 反复重启。回滚模块要先删 `store/crashes.db`。

**包索引要能在鉴权失败时重载一次**（`authenticate_uid`，`PACKAGE_RELOAD_INTERVAL_MS` 限频）。索引在启动时建好，而管理器重装/更新会把 APK 挪到 `/data/app` 下新的随机目录，长跑的守护进程会一直拒绝真正的管理器、界面上表现为「未连接」——每次更新管理器都会遇到，不能指望重启。限频是防任何 app 靠反复连接让守护进程枚举全部包。

**保留策略只删正文与明细行，不动 `occurrence`**：聚合值在写入时维护、读取时不算。分组页因此要说明「另有 N 次已超出保留上限」，否则两个数字看起来自相矛盾。

### 协议与鉴权


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lingqiqi5211/CrashCatcher](https://github.com/lingqiqi5211/CrashCatcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
