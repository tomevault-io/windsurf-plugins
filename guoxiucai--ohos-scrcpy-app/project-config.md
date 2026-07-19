---
trigger: always_on
description: OpenHarmony 设备投屏 + HDC 调试工具。Flutter 桌面客户端通过 `hdc fport` 端口转发与设备上常驻的 OpenHarmony 系统服务通信，实时镜像屏幕并提供触控注入、HAP 安装、HAP卸载、亮度/音量、模拟终端、功能键模拟等控制功能。
---

# CLAUDE.md

OpenHarmony 设备投屏 + HDC 调试工具。Flutter 桌面客户端通过 `hdc fport` 端口转发与设备上常驻的 OpenHarmony 系统服务通信，实时镜像屏幕并提供触控注入、HAP 安装、HAP卸载、亮度/音量、模拟终端、功能键模拟等控制功能。

## 沟通规则

- **永远使用简体中文沟通**，包括所有回复、注释、提交信息等。

## 仓库结构

```
ohos-scrcpy-app/
├── scrcpy_server/             # OpenHarmony 系统应用（ServiceExtension 主体）
│   ├── AppScope/app.json5     # bundleName=com.ohos.scrcpy.server，配置 keepAlive
│   └── entry/src/main/
│       ├── module.json5       # mainElement=ScrcpyService（不是 EntryAbility）
│       ├── ets/
│       │   ├── scrcpyservice/ # 业务全部在此：ScrcpyService、TcpServer、ScreenPipeline、InputInjector、Protocol
│       │   ├── entryability/  # 仅做调试 UI 入口，不写业务
│       │   └── pages/         # 调试页：服务状态、监听端口
│       └── resources/base/element/string.json
├── scrcpy_client_flutter/     # Flutter 桌面客户端（macOS/Windows，Linux 后续）
│   ├── lib/
│   │   ├── hdc/               # 调用系统 hdc CLI（list/fport/install/shell）
│   │   ├── net/               # 协议编解码 + TCP 客户端
│   │   ├── decoder/           # MethodChannel 'scrcpy/decoder' 抽象
│   │   ├── state/app_state.dart
│   │   └── ui/                # top_bar / mirror_view / sidebar / split_view
│   ├── macos/Runner/VideoDecoderPlugin.swift   # VideoToolbox + FlutterTexture
│   ├── windows/runner/video_decoder_plugin.*   # MFT stub，待补
│   └── scripts/               # package_mac.sh (dmg) / package_win.ps1 (Inno Setup)
└── docs/design.md         # 项目设计文档
```

## 关键技术决策（已与用户对齐，勿擅自改动）

- **HDC 调用方式**：客户端 `Process.run` 调系统 `hdc` CLI，优先内置 hdc 二进制文件，其次本机 hdc PATH。
- **视频解码**：平台原生硬解（macOS VideoToolbox / Windows MediaFoundation），通过 MethodChannel + Flutter Texture 上屏。**不**用 media_kit / fvp / ffi+ffmpeg。
- **传输通道**：仅 `hdc fport tcp:<pcPort> tcp:53535`，USB/网络 hdc 共用同一通道，**不**做设备 LAN 直连。
- **服务端保活与自启**：完全交给 `app.json5` 中已配置的 `keepAlive`，**不**写 BootReceiver/StaticSubscriber，**不**申请 `MANAGE_APP_KEEP_ALIVE` / `RECEIVER_STARTUP_COMPLETED`。
- **mainElement**：`module.json5` 的 `mainElement` 必须指向 `ScrcpyService`（ServiceExtensionAbility），UIAbility 仅作调试 UI 入口，不放业务逻辑。
- **API 区间**：`compatibleSdkVersion: 15`，`compileSdkVersion: 20`。

## 通信协议

帧格式：`4B type | 4B length(BE) | payload`

| type | 名称       | 方向    | payload                                          |
|------|------------|---------|--------------------------------------------------|
| 0x01 | 心跳       | 双向    | 空 / 时间戳 8B                                  |
| 0x02 | 视频配置   | S→C     | width(4) height(4) fps(4) spsLen(2) sps ppsLen(2) pps |
| 0x03 | 视频帧     | S→C     | flags(1, bit0=keyframe) + pts(8) + Annex-B NAL  |
| 0x10 | 控制       | C→S     | subType(1) + body                                |
| 0x20 | 设备状态   | S→C     | subType(1) + body                                |

`ControlSubType`：触摸 0x01-0x03（body=x(4) y(4) pointerId(2)）、KEY 0x10、VOL_UP/DOWN 0x20/0x21、BR_UP/DOWN 0x22/0x23。

服务端在设备 `127.0.0.1:53535` listen；客户端 `hdc -t <sn> fport tcp:0 tcp:53535` 拿到 PC 侧端口后连接。

## 构建命令

服务端（OpenHarmony hap）：

```bash
cd scrcpy_server && \
  /Applications/DevEco-Studio.app/Contents/tools/node/bin/node \
  /Applications/DevEco-Studio.app/Contents/tools/hvigor/bin/hvigorw.js \
  clean --mode module -p product=default assembleHap \
  --analyze=normal --parallel --incremental --daemon
```

客户端 Flutter（Flutter 3.22.1）：

```bash
cd scrcpy_client_flutter
flutter pub get
flutter analyze
flutter run -d macos                 # 运行调试
flutter run -d windows    
flutter build macos --debug          # 调试构建
flutter build macos --release        # 然后 scripts/package_mac.sh 出 .dmg
flutter build windows -- # Windows 端要在 Windows 机器上执行
flutter build windows --release # Windows 端要在 Windows 机器上执行


# 打包
# 先cd 到 scrcpy_client_flutter目录
bash scripts/package_mac.sh   #mac打包（签名 + 公证）
powershell -ExecutionPolicy Bypass -File scripts\package_win.ps1  #windows打包（Inno Setup + 可选签名）

```

实机联调步骤：

1. 服务端 `hdc install -r entry-default-signed.hap`
2. PC `hdc -t <sn> fport tcp:5005 tcp:53535`，`nc 127.0.0.1 5005` 看是否能收到字节流
3. 客户端 `flutter run -d macos` → 选设备 → 连接

## ArkTS 严格语法注意点（写服务端代码时常踩）

- 禁止内联对象类型：`Array<{type, payload}>` → 抽 `interface`。
- 禁止内联对象字面量：`{a, b}` → 必须有目标 interface/class，并且**字段名要写全**（不能用 ES2015 的 shorthand）。
- 禁止 `any/unknown`：所有变量带显式类型，包括 `for...of`/迭代器场景。
- `Map.values()` 不要 `for...of`，用 `.forEach`。
- 模块 import 注意：
  - `wantAgent` 用 `import wantAgent from '@ohos.app.ability.wantAgent'`（不要从 `@kit.AbilityKit` 解构，会拿到旧版）。
  - `WantAgentInfo` 字段用新版 `actionType` / `actionFlags`，不是旧版 `operationType` / `wantAgentFlags`。
  - `StaticSubscriberExtensionAbility` 不要再用了（项目不需要 BOOT 监听）。
- 系统应用专属 API：`@ohos.multimodalInput.inputEventClient.injectTouchEvent({ touchEvent })`，`TouchEvent` 字段必须从 `@ohos.multimodalInput.touchEvent` import (`Action`, `SourceType`, `Touch`, `ToolType`)。
- `media.VideoEncoder` / `screenCapture` 命名空间在 ArkTS 12 不存在；屏幕采集 + H264 编码要走 OH Native AVCodec (NAPI)，当前 `ScreenPipeline.ets` 是 stub。
- **NAPI 截屏 API**：`startCapture(cfg)` 创建并启动截屏；`stopCapture()` 销毁截屏与编码器、释放系统资源。无客户端时**必须** stopCapture，不许常驻占用屏幕采集/编码资源。重连时重新 startCapture。

## 平台沙盒

macOS entitlements 已关闭 `com.apple.security.app-sandbox`（设为 `false`），以允许 `Process.run('hdc')`、出站 TCP 与本地端口监听。改 entitlements 时记得 `DebugProfile.entitlements` 与 `Release.entitlements` 两份都改。

## 状态摘要


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guoxiucai/ohos-scrcpy-app](https://github.com/guoxiucai/ohos-scrcpy-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
