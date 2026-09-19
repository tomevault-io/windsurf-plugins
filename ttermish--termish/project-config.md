---
trigger: always_on
description: Kotlin Multiplatform mobile SSH client (Android / iOS). Pure-Kotlin
---

# AGENTS.md — Termish

Kotlin Multiplatform mobile SSH client (Android / iOS). Pure-Kotlin
terminal emulator + Compose Multiplatform shared UI; the SSH transport is swapped
per platform against battle-tested engines: sshj + BouncyCastle on JVM, libssh2 +
OpenSSL on iOS, and a pure-Kotlin Mosh client (`dev.termish.mosh`).
Stack: Kotlin 2.1.21 · Compose Multiplatform 1.8.1 · AGP 8.9.2 · Gradle 8.14.2.

The README introduces the app; full build/test docs live in `CONTRIBUTING.md`
(Build & Test). All workflow tasks are
defined in the root `build.gradle.kts`; `Makefile` targets are thin aliases — CI
reuses the same Gradle tasks, so `make X` and `./gradlew <task>` are equivalent.

## Common commands

```bash
make run                 # build + install debug APK on device/emulator and launch
make test                # unit tests (crypto RFC vectors + terminal emulator)
make test-integration    # transport integration tests (auto-starts local sshd)
make lint                # Android lint
make release             # signed release APK + AAB (requires .env signing secrets)
make bump V=1.0.1        # bump Android and iOS versions (preview: DRY=1)
make ios-native          # one-time cross-compile OpenSSL + libssh2 → iosApp/native/
make ios-framework       # Kotlin framework (simulator + device debug)
```

**Android 模拟器必须用 `-gpu host` 启动**：本机是 KVM 虚拟机，直通了 Intel
Arc A380（`/dev/dri/renderD129`，headless EGL/Vulkan 已验证可用）。禁止
`-gpu swiftshader_indirect` / `lavapipe` 软渲染——软渲染把 GPU 的活全压给
CPU，模拟器常年 200%+ CPU、机器发热。AVD `termish` 的 `hw.gpu.mode` 已设为
`host`；启动命令不要再显式传软渲染参数（`-gpu` 参数会覆盖 AVD 配置）。

Test sshd: `./scripts/test-sshd.sh` (127.0.0.1:22222, generates ephemeral ed25519 keys).
Demo server (screenshots / herdr+pi): Docker container `termish-demo` on the dev
machine — credentials & ports are kept in local test assets (`.aiadb/`, gitignored),
not in this repo. Android emulator reaches the host at `10.0.2.2`; iOS simulator
(shares host network) at `127.0.0.1` — never `10.0.2.2` on iOS.

## Architecture & conventions

- **`commonMain/term/` is a pure-Kotlin, zero-platform-dependency terminal
  emulator** (buffer / state machine / color / selection) — never pull platform
  APIs or Compose dependencies into it
- **`commonMain/mosh/` protocol layer is term-free**: `MoshTransport` /
  `UserStream` / `Fragmentation` / `Messages` / `Ocb` / `Aes` / `MoshCrypto` /
  `KmpMoshSession` must never `import dev.termish.term` — only the shadow
  layer (`ShadowTerminal`, `PredictionLayer`) may use the emulator (a mosh
  client must mirror the server framebuffer; reusing our own emulator is
  deliberate, not a shortcut). Keep this boundary: it's what makes the
  protocol layer extractable as a standalone library
- Platform code only lives behind expect/actual seams: `ssh/SshSession`, `util/`,
  `data/SecretStore`; `androidMain/` owns the Android JVM transport engine
- The input pipeline treats **IME composing text as a first-class citizen**:
  composing text never reaches the wire, only committed text is diffed, and
  backspace semantics are split between composing/committed — read the existing
  implementation in `TerminalScreen` / `TerminalView` before touching input logic
- UI design tokens live in `ui/theme/` (Dimens, palettes) — no ad-hoc dp/alpha
  literals in new UI code
- User-facing strings go through `AppStrings` (Chinese + English), never hardcoded
- **Code style**: Kotlin official style, `import` 短名（禁止全限定名调用）、
  import 按字母序（`.editorconfig` 基线 + ktlint 已接入，`make lint-kt` 检查/`ktlintFormat` 自动修）
- New screenshots in `docs/screenshots/` use `<topic>-{en,zh}.png` or `.webp`
  naming; en = English UI, zh = Chinese UI, every topic has a matching pair.
  The README gallery shows the Herdr terminal (including remote screen), Hosts,
  optional Agent chat and SFTP as equally sized iOS device captures. Reuse
  suitable public demo screenshots, including assets in `docs/appstore/`, and
  keep personal accounts and private host details out of the selection.

## 文档维护

- `README.md`（英文）与 `README.zh-CN.md`（中文）同步维护结构与事实性内容
- `docs/` 为面向贡献者的开发文档，正文中文、开头英文摘要
- `term/`、`mosh/`、输入交互的行为变更同步更新 `docs/terminal-emulator.md`、
  `docs/mosh.md`、`docs/input-pipeline.md` 中对应的说明
- 项目原始源码与文档以 MIT 开源；第三方版权声明及 `NOTICE`、
  `LICENSES/` 和资源内附带的许可证必须保留

## Development workflow（开发工作流）

分层验证，由快到慢，逐级上升；**日常迭代全走 debug + 模拟器，release 只在功能里程碑/准备验收时构建**：

```bash
# ① 单元测试（秒级）——改动涉及 term/、mosh/、crypto/、逻辑层时必跑
./gradlew :composeApp:testDebugUnitTest

# ② debug 构建 + 模拟器安装启动（分钟级）——日常迭代主力
make run

# ③ 集成测试（分钟级）——传输层（sshj/libssh2/mosh/SFTP）改动时必跑
make test-integration

# ④ 真机抽查（手动）——模拟器测不了的点
#    - IME 中文/日文输入（模拟器常无真实输入法，CJK 组合态管线必须真机）
#    - Mosh 漫游（WiFi↔蜂窝切换续传）、后台保活、断线重连
#    - 性能 / 电池

# ⑤ 签名 release（分钟级）——功能完成/准备验收时构建
make release    # 产物 composeApp/build/outputs/{apk,bundle}/release/
```

规则：

- **release 不是每次改动的必做项**——R8 混淆/资源收缩/签名问题在里程碑验证时暴露即可，日常被 debug 循环拖慢
- **有模拟器时模拟器安装验证优先**（`make run`）；真机按上面 ④ 的点抽查
- **提交与发版必须等用户明确确认**：本地改动攒批即可，**不得擅自** `git commit` / `make bump` / 打 tag / `git push`（历史教训：用户反馈 bug 后直接推送 GitHub 是不被接受的）。完成一个功能块后：本地验证（①/②）+ 汇报结果，**等用户说「提交/发版」再动 git**；中途防丢失用 `git stash` 或 WIP commit（WIP 不 push）
- 改动涉及 `term/` 或 `mosh/` 时先跑 ① 再上模拟器（秒级反馈，别浪费模拟器循环）

## Testing discipline

- Any change to `term/` (escape sequences, buffer, wide-char behavior) → add a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ttermish/termish](https://github.com/ttermish/termish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
