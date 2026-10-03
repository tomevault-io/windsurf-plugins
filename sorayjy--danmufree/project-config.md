---
trigger: always_on
description: DanmuFree 是用户**从零自建**的 B站 / 抖音直播间弹幕桌面客户端（Windows / WPF / .NET 8），作为商业弹幕助手的自用替代品。**双平台、个人自用**：B站为主，抖音为第二数据源，共用同一套展示管道。
---

# CLAUDE.md — DanmuFree

## 项目背景
DanmuFree 是用户**从零自建**的 B站 / 抖音直播间弹幕桌面客户端（Windows / WPF / .NET 8），作为商业弹幕助手的自用替代品。**双平台、个人自用**：B站为主，抖音为第二数据源，共用同一套展示管道。

核心设计决策：**协议层完全自写、零第三方直播库**。B站协议会频繁变动（本项目周期内已踩到 `getRoomInfoOld` 弃用、op3 在线失真、登录 cookie 改走 `Set-Cookie`、`INTERACT_WORD_V2` 把交互信息塞进 `data.pb` protobuf 等）；抖音 WS 握手要 X-Bogus 签名（webmssdk 反调试 JS）——依赖第三方库会腐烂；自写 + 单测，协议变了改一处即可（抖音签名换 `sign/sign.js` 即可）。

## 仓库布局
- `src/DanmuFree.Core/`（net8.0，纯逻辑，零第三方）
  - `Protocol/`：FrameCodec（16 字节大端帧）、PayloadCodec（zlib/brotli 解压）、PacketDecoder（递归解码）、MessageParser（cmd→RichMessage；DANMU_MSG 勋章；**INTERACT_WORD_V2 解码 `data.pb` protobuf**：顶层 f2=uname、f5=msg_type；**SEND_GIFT_V2 解码 `data.pb` protobuf（2026-09 修，B站 2026-07 灰度的新礼物协议，新旧并存按礼物分流——「灯牌显示人气票消失」的根因）**：顶层 f2=uname、f10=gift_list[]（每项 f2=gift_name、f3=num，一帧可多条）；`Parse` 单条契约定格为首条包装、新 `ParseAll` 批量契约（client 收包循环已切换），产出与 SEND_GIFT 同构（Gift + Extra「{名} x{N}」）显示/聚合朗读全链路零改动；**GUARD_BUY 上舰（舰长/提督/总督）按礼物路由**——字段 `username`/`gift_name`/`num`，与 SEND_GIFT 的 `uname`/`giftName` 不同）、RoomResolver（`getInfoByRoom` 解析真实 room_id；**`getDanmuInfo` 带 WBI 签名 `w_rid`**）、**WbiSign**（B站 WBI `w_rid` 签名纯逻辑：`GetMixinKey`/`Sign` + 64 项 `MixinKeyEncTab` 重排，可单测）、StatsService（统计轮询）。MessageParser 另解 **B站 WS 实时统计推送（2026-09 接入）：`ONLINE_RANK_COUNT`（真实在线人数，2~5s 一条）→ 新枚举 `RealOnlineCount`、`WATCHED_CHANGE`（看过累计）→ 新枚举 `WatchedCount`**，数字放 Extra（与 OnlineCount 契约一致）。注意与 `OnlineCount`（op3 心跳 / ONLINE_GUEST_COUNT，人气口径，实测 7777 op3 恒 1）区分——旧通道 VM 收到即丢、保持弃用；VM 收到新消息置 `_hasRealOnline`/`_hasRealWatched` 后 StatsService 60s 轮询不再覆盖对应字段（不推这两条的房间回落轮询的人气值）。实测 7777：count≈3.5k（与 `queryContributionRank?type=online_rank` 的 `data.count` 同源）vs getInfoByRoom `online`=人气 27 万 vs op3=1。
  - `Client/`：BilibiliDanmuClient（WebSocket 连接 / 认证 / 心跳 / 收包 / 指数退避重连）；**DouyinDanmuClient**（抖音 WS：房间解析→node 签名→握手→心跳→收帧解码→ack→RichMessage→重连，公开面与 B站 client 一致）。
  - `Protocol/`（抖音新增）：**DouyinProto**（手写 protobuf varint：PushFrame/Response/Message/Chat/Gift/RoomUserSeq 解码 + BuildAck/BuildHeartbeat）、**DouyinSign**（纯 C# 算签名素材：13 字段 param→md5=X-MS-STUB + BuildConnectUrl[域名 `-ws-web-`] + `IDouyinSigner` 抽象）、**DouyinMapper**（method→RichMessage）、**DouyinRoomResolver**（主页抠 ttwid + enter 接口取真实 room_id，**不需 a_bogus**）。
  - `Login/`：LoginService（扫码登录，**优先从 `Set-Cookie` 响应头提取 cookie**，`data.url` query 作兜底）、QrStatus、QrInfo。
  - `Models/`：RichMessage（record）、MessageType。
  - `Tts/`：**ITtsClient**（`SynthesizeAsync→Stream`，**契约放宽为 WAV 或 MP3**）、**EdgeTtsClient**（**Core、`System.Net.WebSockets`、零第三方**；clean-room 实现 Edge Read Aloud 协议，返 MP3；`BuildSecMsGec`/`BuildSsml`/`SpeedToRate` 纯逻辑可单测；`SupportedVoices` 14 个中文音色 `EdgeVoice(Id,Display)`）、**GptSoVitsClient**（HTTP GET+query→wav）、**TtsOptions/TtsReadFlags**、**TtsTextBuilder**（按 flags 转朗读文本；**SC 是事件型恒带用户名/金额：「xx 送了 30 元的 SC，内容」**——价格取 Extra「¥30」，不受「读用户名」开关影响）、**GiftReadAggregator**（礼物连送聚合，纯逻辑）、**ReplyRule**（定向回复：`ReplyAction`(念文字/播音频) + `ReplyRuleMatcher.MatchFirst`——规则按序**首条命中即停**、空关键词/空载荷/**禁用(`Enabled=false`)**规则跳过；子串匹配忽略大小写。纯逻辑可单测）。
- `src/DanmuFree.App/`（net8.0-windows，WPF）
  - `Views/`：DanmuWindow（弹幕显示，纯展示窗；顶部常驻统计条 在线/看过/赞，`ShowStats` 可整条收起）、NotifyWindow（进场/关注独立窗，纯展示窗）、ControlWindow（**常驻**控制/设置，任务栏可最小化，标题栏拖动）、LoginDialog（扫码）。
  - `Controls/`：**OutlineHost**（文字描边容器，Decorator：代码自建 8 层嵌套 Grid 各挂 BlurRadius=0 的 DropShadowEffect、8 方向硬投影叠加成描边环；`Stroke`(Color)/`Thickness`(double) 两 DP 的 **ChangedCallback 直改效果属性**；Thickness=0 时效果置 null（零开销）。弹幕/通知窗模板各包一层，设置独立；描边色可点色块开 Win32 颜色盘（WinForms ColorDialog，csproj `UseWindowsForms` 仅为此一处、并 `<Using Remove>` 掉其隐式 global using 防 CS0104 撞名）。
  - `ViewModels/`：DanmuViewModel（MVVM，CommunityToolkit.Mvvm；含 EffectiveTopmost/EffectiveOpacity / EffectiveNotifyTopmost/EffectiveNotifyOpacity 等「进悬浮自动生效、退出还原」的派生属性；两个集合 Messages/NotifyMessages + 两个 pump）、ReplyRuleViewModel（定向回复规则可编辑行：Action 字符串 "text"/"sound"，`ToRule()` 转 Core 匹配用 record）。
  - `Services/`：AppSettings、SettingsService（%AppData%/DanmuFree/settings.json）、FileLogger、UiBatchPump（Channel→100ms 批量刷 UI）、QrImageRenderer、**WindowClickThrough**（Win32 `WS_EX_TRANSPARENT` 鼠标穿透）、**DouyinSigner**（实现 `IDouyinSigner`：md5→node 子进程跑 `sign/sign_runner.js`→X-Bogus）、**TtsSpeaker**（NAudio 串行播放；队列条目 **`TtsItem`（文本/本地音频二选一**——播音频直接 `File.OpenRead` 复用同一条嗅探路径）；嗅探音频头 RIFF→`WaveFileReader`，否则→`Mp3FileReader` 解码 MP3）、**SystemSpeechTtsClient**（App 层 SAPI 内置引擎，`ListVoiceNames()` 枚举系统音色）、**GiftTtsPump**（礼物朗读 debounce）。
  - `sign/`：抖音签名 sidecar —— `sign.js`（webmssdk 1.0.0.53 + jsdom 补环境，saermart 逆向）+ `sign_runner.js` + `package.json` + `node_modules/jsdom`（csproj `Content Include sign\**` 随 exe 分发；node_modules 本地存在即复制、gitignore 不入库）。
  - `Converters/`：MessageTypeToBrush、OpacityToBackground、TimeIfShown、**StringEqualsConverter**（朗读 TAB 按 TtsEngine 切引擎控件可见 / RadioButton 双向）。
- `tests/DanmuFree.Tests/`（net8.0）：Core 层单测 + `Helpers/FakeHttpHandler`（支持 `.When(url, json)` / `.WithSetCookies(url, ...)`）。

## 架构要点

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SoraYjy/DanmuFree](https://github.com/SoraYjy/DanmuFree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
