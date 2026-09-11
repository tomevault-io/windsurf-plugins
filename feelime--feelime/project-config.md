---
trigger: always_on
description: 给 AI agent（和新同事）的工程约定与操作手册。读完本文件即可安全地改代码、
---

# AGENTS.md — AI 协作指南

给 AI agent（和新同事）的工程约定与操作手册。读完本文件即可安全地改代码、
跑门禁、发版本；产品语义的权威出处见 [docs/README.md](docs/README.md)。

## 项目一句话

Feelime 是一个 Android 离线输入法：Kotlin `InputMethodService` + WebView
渲染的 HTML 键盘（`app/src/main/assets/keyboard/`），通过 token 鉴权的
JS Bridge 调用 `InputConnection`；离线语音（sherpa-onnx 流式 + 整句
纠错双通道）与 librime 中文引擎。

## 仓库布局

```
app/src/main/assets/keyboard/   键盘前端（index.html / keyboard.js / keyboard.css / VERSION）
app/src/main/assets/settings/   完整设置页（HTML，不参与键盘热更新）
app/src/main/java/com/feelime/ime/   IME 服务、Bridge、引擎、面板、更新（Kotlin）
app/src/main/res/               图标与资源（ic_launcher.xml 与 docs/branding 同源）
scripts/verify/                 自动化验证套件（mock / css lint / 设备门禁）
scripts/research/               ASR 回归门禁、Hunspell 前缀索引生成等工具
docs/                           文档（见 docs/README.md 的主题索引）
tools/                          真实源码预览（keyboard-preview / settings-preview）
spikes/                         独立 Gradle 工程：librime 引擎冒烟、ASR 基线
test-fixtures/                  验证测试数据（旧代键盘源码、双拼键位表）
scripts/research/               native 词典引擎 pinned 构建管线与验证脚本
third_party/                    许可证清单与来源闭包
```

## 常用命令

```bash
./scripts/setup-assets.sh          # 首次：下载 sherpa-onnx AAR/模型（SHA-256 校验，
                                   #   安装到 ~/.config/feelime，全 worktree 共享）
ANDROID_HOME=… ./gradlew :app:assembleDirectDebug
                                   # 构建（产物 app/build/outputs/apk/direct/debug/）
./scripts/push-keyboard.sh         # 键盘热更新：只推 HTML/CSS/JS，不重装 APK
adb install -r app/build/outputs/apk/direct/debug/app-direct-debug.apk
```

## 本机构建环境（不入仓库）

仓库面向开源，任何主机名、本机路径、签名凭据都不得入库；它们统一放在
每台机器的 `~/.config/feelime/`，所有 worktree/clone 共享一份：

```
~/.config/feelime/
├── env.sh      # KEY=VALUE 行、'#' 注释；键位表见下
├── models/     # ASR 模型树（setup-assets.sh 的安装目标，一次下载处处可用）
└── android/    # 大件构建资产（sherpa-onnx AAR）
```

加载语义（`scripts/feelime-env.sh` 与 `app/build.gradle.kts` 一致）：
文件被**解析**（不执行），只填充**未设置**的 `FEELIME_*` 变量——显式环境
变量永远优先。`env.sh` 键位（全部可选）：

| 键 | 用途 |
| --- | --- |
| `FEELIME_ANDROID_HOME` | Android SDK |
| `FEELIME_NDK_HOME` | NDK（pinned native 管线用） |
| `FEELIME_AAPT2` | aapt2 覆盖路径（x86_64 主机必配，见验证节） |
| `FEELIME_MODELS_DIR` | 共享模型树位置（默认 `~/.config/feelime/models`） |
| `FEELIME_ANDROID_DIR` | 共享 AAR 位置（默认 `~/.config/feelime/android`） |
| `FEELIME_ADB_SERIAL` | 门禁默认测试设备 |
| `FEELIME_ASR_FIXTURE` | ASR 门禁录音（16 kHz wav，自备、不入库） |
| `FEELIME_PREVIEW_ROOT` / `_URL` | demo 预览发布目录 / 访问链接前缀（`tools/deploy-preview.sh`，见外观变更流程） |
| `FEELIME_BUILDER_SSH` / `_DIR` / `_SDK` | 远端构建机（JVM 套件） |
| `FEELIME_DEBUG_KEYSTORE` / `_STORE_PASSWORD` / `_KEY_ALIAS` / `_KEY_PASSWORD` | 调试签名（缺省用 Android 默认 debug.keystore） |
| `FEELIME_RELEASE_KEYSTORE` / `_STORE_PASSWORD` / `_KEY_ALIAS` / `_KEY_PASSWORD` | 发布签名（正式私钥离线保存） |

**调试签名必须跨机构建一致**：默认 debug.keystore 每台机器不同，签名
不一致的包互相「升级」会被系统判定为不同应用、强制卸载重装——用户设置、
常用词、词库全部丢失（输入法还要重新启用）。所有参与构建的机器把同一份
`debug.keystore` 放在 `~/.config/feelime/` 并在 `env.sh` 配置
`FEELIME_DEBUG_KEYSTORE` 指向它。

模型目录解析优先级（`FEELIME_MODELS_DIR`）→ `~/.config/feelime/models` →
旧版仓库内 `app/src/modelAssets/full`（向后兼容，逐步废弃）。

**无模型 APK 的设置页下载验证**：thin/无模型 APK 在设置页走 ModelStore
下载。当前产品的下载源由设置页「模型源」选择驱动（`hf_mirror` /
`official` / `custom`），构建时嵌入的 `models/dev-urls.json` 镜像前缀
**只在设备端源选择为空时生效**。`custom` 接受 http/https（下载完整性
由 manifest 的 SHA-256 逐文件校验兜底，transport 不是信任边界）；custom
base 按 HuggingFace 仓库结构拼接（`base + <owner>/<repo>/resolve/main/ +
downloadPath`），自建源需镜像该目录布局：

```bash
# 生成本机 web 根并起 Range-aware 临时 server（设备下载器用 Range 续传，
# `python3 -m http.server` 不支持 Range 会损坏 .part，勿直接用）
scripts/dev-models.sh --serve <设备可达的主机或IP> 8765
# 之后构建 thin 包：-PfeelimeModels=thin -PfeelimeDevModels=true
# 注意：设备端「模型源」若已固定为 hf_mirror/official，需在设置页改回
# 或清除，dev 前缀才会生效；hf-mirror 上的部分模型对象与发布归档不一致
# （如 streaming encoder），固定走它必然校验失败
```

## 验证（改动的验收标准）

方法、套件清单与平台怪癖的完整说明见
[docs/testing/verification.md](docs/testing/verification.md)。速查：

```bash
# 本地三层（无设备，秒级）——任何改动至少过这三层
node scripts/verify/css_lint.js
node scripts/verify/mock_bridge_tests.js
ANDROID_HOME=… ./gradlew testDebugUnitTest

# 设备层（环境变量注入；run-all 第一步核对装机 APK SHA）
export FEELIME_ADB_SERIAL=<serial>
export FEELIME_VERIFY_APK=$PWD/app/build/outputs/apk/direct/debug/app-direct-debug.apk
export FEELIME_ASR_FIXTURE=$PWD/mixed-zh-en.wav   # 本地自备 16kHz wav，不入库
# 可选：FEELIME_BUILDER_SSH 可把 JVM 套件放到远端跑；仅本机构建时无需设置
bash scripts/verify/run-all.sh    # 全量门禁
```

## 词典引擎数据（engine-data）

`app/src/main/assets/engine-data/`（rime/hunspell/mozc 数据，约 52MB）随仓库
分发，构建时 `checkEngineArtifacts` 按 `third_party/manifest.json` 与
`MANIFEST.json` 的 SHA-256 逐文件核对。再生成走 pinned 管线（输入全部是
上游开源归档，哈希钉死）：

```bash
./scripts/research/fetch-native-engine-inputs.sh ~/tmp/feelime-native-inputs
# NDK r29 解压到任意目录后：
FEELIME_NATIVE_INPUTS=… FEELIME_NATIVE_NDK=… FEELIME_NATIVE_WORK=… \
  bash scripts/research/build-cmake-native-engines.sh   # so + rime-data
# mozc.data 另由 scripts/research/build-mozc-android.sh 构建
```

## 键盘热更包签名身份

正式 Ed25519 私钥离线保存，仓库只有公钥（`ReleaseKeys`）。
`app/src/test/resources/keyboard-update-signed.zip` 由一次性测试身份
`feelime-roundtrip-test` 签名（私钥 PEM 就在测试资源里，只用于打包器↔校验器
的往返测试，不在生产信任集）。重打：

```bash
FEELIME_SIGNING_KEY=app/src/test/resources/keyboard-update-roundtrip-test-ed25519.pem \
  FEELIME_KEY_ID=feelime-roundtrip-test scripts/package-keyboard.sh \
  app/src/test/resources/keyboard-update-signed.zip
```

约定：**先模拟器收敛、再真机终验**；改可见语义先 grep

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feelime/feelime](https://github.com/feelime/feelime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
