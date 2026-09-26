---
trigger: always_on
description: 本文件适用于整个仓库。若某个子目录中存在更具体的 `AGENTS.md`，则该子目录优先遵循更具体的文件。
---

# AGENTS.md

## 适用范围

本文件适用于整个仓库。若某个子目录中存在更具体的 `AGENTS.md`，则该子目录优先遵循更具体的文件。

项目暂定名：**DesktopDlnaCast**。

本项目是一个 Windows 桌面 GUI 应用，用于：

1. 捕获 Windows 显示器或指定应用窗口；
2. 可选捕获系统正在播放的声音；
3. 将画面和声音实时编码成智能电视易于兼容的直播媒体流；
4. 在局域网中通过 HTTP 提供该媒体流；
5. 使用 UPnP/DLNA 控制电视等 Digital Media Renderer（DMR）播放该流。

本项目的准确定位是：

> **通过 DLNA 播放 Windows 桌面的实时直播流。**

它不是 Miracast 实现，不是虚拟显示器驱动，也不是远程桌面协议。

---

# 1. 产品目标

用户应当能够在 GUI 中完成以下操作：

- 搜索当前局域网内的 DLNA/UPnP MediaRenderer 设备；
- 查看设备名称、厂商、型号和 IP 地址；
- 选择某一块显示器或某个应用窗口；
- 选择是否包含鼠标指针；
- 选择是否包含 Windows 系统声音；
- 选择兼容、标准或自定义画质；
- 选择自动、连续 MPEG-TS 或 HLS 输出模式；
- 用已知可播放的测试片段验证电视的 DLNA 播放链路；
- 一键开始和停止桌面投屏；
- 在失败时查看并导出足够详细的诊断信息。

视频数据链路：

```text
Windows 显示器或窗口
    ↓
Windows.Graphics.Capture
    ↓
D3D11 缩放、裁剪、色彩转换
    ↓
H.264 实时编码
```

音频数据链路：

```text
Windows 输出设备
    ↓
WASAPI Loopback
    ↓
重采样与格式转换
    ↓
AAC-LC 实时编码
```

媒体输出链路：

```text
H.264 + AAC
    ↓
MPEG-TS 复用
    ├─ 连续 HTTP MPEG-TS，默认低延迟模式
    └─ HLS，电视兼容回退模式
    ↓
电视通过 HTTP 主动拉流
```

DLNA 控制链路：

```text
SSDP 发现
    ↓
获取 Device Description XML
    ↓
解析 AVTransport / ConnectionManager
    ↓
GetProtocolInfo 能力探测
    ↓
SetAVTransportURI
    ↓
Play
```

MVP 的自动化验收目标是仓库内自带的、提供标准 UPnP AVTransport 服务的测试 Receiver。它必须像真实 DMR 一样完成 SSDP 响应、接收控制命令、主动拉取并验证媒体流。真实电视或盒子只用于发布前的人工兼容性抽查，不得成为 AI 开发、CI 或里程碑推进的必备条件。

---

# 2. 明确不做的事项

除非后续 issue 明确修改范围，否则不要实现：

- Miracast；
- Wi-Fi Direct；
- Google Cast / Chromecast；
- AirPlay；
- Lelink、乐联、Cast+ 等私有投屏协议；
- WebRTC；
- Windows 虚拟显示器或 Indirect Display Driver；
- 远程鼠标、键盘、触控或手柄输入；
- DRM 绕过；
- 受保护视频捕获绕过；
- HDCP 规避；
- 云端中继；
- 公网直播；
- NAT 穿透或路由器端口映射；
- 完整的 DLNA MediaServer；
- ContentDirectory 服务；
- 默认把桌面录制到硬盘；
- MVP 阶段同时投多台电视；
- MVP 阶段默认使用 HEVC/H.265；
- 裸 H.264 与裸 PCM 的私有混流；
- 游戏级低延迟；
- 对低于 200 ms 延迟作出承诺。

电视在本项目中是一个网络媒体播放器，而不是真正的 Windows 第二显示器。

---

# 3. 初始平台范围

初始支持目标：

- Windows 11；
- x64；
- 当前受支持的 Visual Studio；
- 当前受支持的 Windows SDK；
- 当前受支持的 .NET SDK；
- 当前受支持的 Windows App SDK。

在不会明显增加复杂度的前提下，托管代码应避免不必要的 x64 假设，为以后支持 Windows on ARM64 保留空间。

初始阶段：

- 不支持 x86；
- 不要在 x64 版本尚未稳定前投入 ARM64 原生媒体核心；
- 不要为了支持过旧的 Windows 版本牺牲架构清晰度。

如果实际实现需要降低最低 Windows 版本，必须先提交设计说明和验证结果。

---

# 4. 固定技术基线

## 4.1 GUI 与托管宿主

使用：

- C#；
- WinUI 3；
- Windows App SDK；
- MVVM；
- `Microsoft.Extensions.DependencyInjection`；
- `Microsoft.Extensions.Logging`；
- Kestrel 嵌入式 HTTP Server。

托管层负责：

- GUI；
- ViewModel；
- 应用配置；
- 投屏会话编排；
- SSDP 发现；
- UPnP Device Description 解析；
- SOAP 控制；
- DIDL-Lite 生成；
- 电视兼容配置；
- HTTP Server 生命周期；
- 日志与诊断导出。

禁止：

- 在 UI 线程执行阻塞网络操作；
- 在 UI 类中运行实时帧处理循环；
- 在 UI 线程调用 `.Result`、`.Wait()`；
- 用隐藏的无生命周期后台任务承载核心媒体流程。

## 4.2 原生媒体核心

实时媒体流水线使用 C++/WinRT 原生 DLL。

原生层负责：

- Windows.Graphics.Capture；
- D3D11 纹理管理；
- GPU 侧缩放和色彩转换；
- WASAPI Loopback；
- Media Foundation H.264 编码；
- Media Foundation AAC 编码；
- FFmpeg `libavformat` 的 MPEG-TS/HLS 复用；
- PTS/DTS；
- 音视频交织；
- 有界队列；
- 关键帧感知的启动缓冲。

托管层通过精简、稳定的 C ABI 使用原生核心。

禁止跨 ABI 暴露：

- C++ STL 容器；
- C++ 异常；
- WinRT 对象；
- COM 接口指针；
- 所有权不明确的裸指针。

建议接口形状：

```c
typedef void* ddc_session_handle;

typedef struct ddc_stream_config {
    int32_t width;
    int32_t height;
    int32_t frame_rate;
    int32_t video_bitrate;
    int32_t audio_bitrate;
    int32_t include_audio;
    int32_t stream_mode;
} ddc_stream_config;

int32_t ddc_session_create(
    const ddc_stream_config* config,
    ddc_session_handle* result);

int32_t ddc_session_start(ddc_session_handle handle);
int32_t ddc_session_stop(ddc_session_handle handle);
void ddc_session_destroy(ddc_session_handle handle);
```

接口可以演进，但必须保持：

- 小；
- 明确；
- 可版本化；
- 可测试；
- 可取消；
- 重复 stop/cleanup 安全；
- 不跨边界抛出异常。


# 5. 推荐仓库结构

```text
DesktopDlnaCast/
├── AGENTS.md
├── README.md
├── LICENSE
├── THIRD_PARTY_NOTICES.md
├── DesktopDlnaCast.sln
├── docs/
│   ├── architecture.md
│   ├── protocol-notes.md
│   ├── compatibility.md
│   └── troubleshooting.md
├── src/
│   ├── DesktopDlnaCast.App/
│   ├── DesktopDlnaCast.Core/
│   ├── DesktopDlnaCast.Upnp/
│   ├── DesktopDlnaCast.Streaming/
│   ├── DesktopDlnaCast.Media.Interop/
│   └── DesktopDlnaCast.Media.Native/
├── tests/
│   ├── DesktopDlnaCast.Core.Tests/
│   ├── DesktopDlnaCast.Upnp.Tests/
│   ├── DesktopDlnaCast.Streaming.Tests/
│   └── DesktopDlnaCast.IntegrationTests/
└── tools/
    ├── MockRenderer/
    └── StreamProbe/
```

## `DesktopDlnaCast.App`

负责：

- WinUI View；
- ViewModel；
- Command；
- 资源字典；
- 本地化资源；
- 用户交互。

不得包含：

- SSDP 协议实现；
- SOAP 底层实现；
- 编码器实现；
- HTTP 流输出实现。

## `DesktopDlnaCast.Core`

负责：

- 投屏会话状态机；
- 用例；
- 接口；
- 配置模型；
- 兼容配置选择；
- 错误模型；
- 会话编排。

## `DesktopDlnaCast.Upnp`

负责：

- SSDP；
- Device Description；
- Service URL 解析；
- SOAP；
- AVTransport；
- ConnectionManager；
- 可选 RenderingControl；
- DIDL-Lite；
- Renderer 设备模型。

## `DesktopDlnaCast.Streaming`

负责：

- Kestrel Host；
- 会话 Token；
- HTTP Endpoint；
- 连续 MPEG-TS 输出；
- HLS Playlist 和 Segment；
- 网卡绑定；
- 目标电视 IP 限制。

## `DesktopDlnaCast.Media.Interop`

负责：

- P/Invoke；
- `SafeHandle`；
- 托管配置到原生配置的转换；
- 原生回调；
- 原生错误翻译。

## `DesktopDlnaCast.Media.Native`

负责：

- 视频捕获；
- 音频捕获；
- D3D11；
- Media Foundation；
- FFmpeg mux；
- 媒体时钟；
- 环形缓冲；
- 原生统计信息。

## `MockRenderer`

实现可重复测试的 UPnP MediaRenderer：接收 SOAP 控制、主动拉取媒体、推进 Transport State，并暴露机器可读的断言与故障注入接口。

## `StreamProbe`

用于检查生成的流，并在开发环境存在 `ffprobe` 时调用它进行额外验证。

协议层、媒体层和 GUI 层必须解耦。

---

# 6. 核心接口与会话状态机

优先采用类似接口：

```csharp
public interface IDlnaDiscoveryService;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liyafe1997/dlna-screen-cast](https://github.com/liyafe1997/dlna-screen-cast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
