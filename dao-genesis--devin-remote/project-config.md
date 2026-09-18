---
trigger: always_on
description: > 本文是后续 Agent 接手本仓前**必读**的本源说明。它不是功能清单（功能见各模块 README），
---

# AGENTS.md · 本源认知与防踩坑（给后续 Agent 看）

> 本文是后续 Agent 接手本仓前**必读**的本源说明。它不是功能清单（功能见各模块 README），
> 而是把历史上反复踩过的坑、容易被「老数据 / 旧叙事」误导的地方一次说清，**正本清源**，
> 防止重蹈覆辙。改动本仓前先读完本文，再读 [`README.md`](README.md) 的「架构全貌 4+4+3」。

---

## 一、本源认知（最重要，先记住）

1. **dao-vsix 才是本源主体。** 所有核心底层（rt-flow 切号 + Devin Cloud 全功能六大板块 +
   本地 HTTP API + 多账号反向注入 + 内网穿透集成 + 多实例）都落在 **`core/dao-vsix`**。
   开发的着重点永远是 dao-vsix，不是 dao-one。
2. **dao-one 只是「dao-vsix + 一个 Proxy Pro 模块」。** `core/dao-one` 是最终归一交付，
   = dao-vsix 本源基座 **折入** dao-proxy-pro 三面板而已，**其余与 dao-vsix 完全一致**。
   不要把 dao-one 当成独立主体去开发；它没有自己的「业务底层」，业务底层全在 dao-vsix。
3. **没有「四合一 / 五合一」这种东西。** 历史对话里残留过 “四合一” 之类的错误叫法，
   那是老数据裹挟，**不代表架构**。准确叫法只有：
   - dao-vsix = **二合一**（切号 + 全功能面板，含本地 API）
   - dao-one = **三合一**（二合一 + Proxy Pro）
   - rt-flow-app（手机 APK）= **七合一**（这是手机端功能计数，与插件无关，别混淆）
   若在代码/文档里再看到 “四合一/五合一” 指代插件，按上面口径就地更正。

---

## 二、六大板块 · 分而治之 · 网页套网页（核心架构）

「Devin Cloud 全功能面板」逻辑上由**六大板块**组成：

| board key | 图标 | 名称 |
|---|---|---|
| `overview` | 🏠 | 主页 / 单账号管理 |
| `switch`   | 🔀 | 切号 / 账号池 |
| `bridge`   | 🌐 | 公网穿透 · DAO Bridge |
| `backups`  | 💬 | 对话备份 |
| `inject`   | 💉 | 反向注入 · 全账号 |
| `mcp`      | 🧩 | MCP 服务器 |
| `github`   | 🐙 | GitHub 纵向板块(独立于 Devin·PAT/组织/迁仓/公私/独立舰队/GitHub MCP 镜像) |

> ⚠️ `github` 是**独立的 GitHub 纵向板块**(与 Devin 账号池毫无关系)——它虽也进 `_solo` 白名单
> 走同一套 solo 单板块渲染, 但语义上**不属于「六大板块」**, 而是并列的第七纵向。它管的是纯
> GitHub 账号/组织/仓库/独立舰队, 切号板块管的是 Devin 账号池, 两者不可混淆。源级护栏见
> `test/board-solo.test.js`(白名单 ≡ 六大板块 + github)。

**分而治之 = 每个板块各开一张独立子网页**，而不是挤在一个全功能面板里靠内部 tab 切换。
承载它的是统一外壳 `/shell`（「归一 Devin Cloud 网页」），它本质是一个**浏览器套浏览器**
（网页套网页）：外壳是一个带标签栏的迷你浏览器，每个板块 / 每个多实例账号页都是其中一张
**平级并排**的标签（各自一个 iframe 子网页）。这样：

- 在 IDE 插件 webview 里能操作的，在任意外部浏览器打开 `/shell` 也能操作；
- 公网用户经 dao-bridge 隧道打开 `/shell` 即可访问同一张归一网页。

### 实现位置（改这块时务必两边对齐）

- **dao-vsix 端（板块 HTML 生产者）**：`core/dao-vsix/src/extension.ts`
  - `getDaoCloudMiddlePanelHtml(st, soloBoard?)`：`soloBoard` 传入某个 board key 时进入
    **单板块模式**——只渲染该板块、并用 `body.solo .sb{display:none}` 隐藏左侧板块导航。
    合法 board key 白名单见该函数顶部 `_solo = [...]`。
  - `setCloudProvider({ buildHtml: (board?) => getDaoCloudMiddlePanelHtml(..., board), ... })`：
    把 board 参数透传给宿主。
- **rt-flow 端（统一外壳 `/shell` 消费者）**：`core/rt-flow/extension.js`
  - `BOARDS` 注册表 + `BOARD_META`：每个板块一张独立 iframe 标签（`board:<key>`）。
  - `mountBoardSolo(html, tab)`：把某板块的 solo HTML 挂成一张独立子网页标签。
  - `/shell` 路由的 `cloudInit` 消息携带 `board` 字段，`cloudInitHtml` 回包回显 `board`。

> ⚠️ dao-vsix 的运行时副本被**捆绑**在 `core/dao-vsix/rtflow/`（独立版用）。改了
> `core/rt-flow/{extension.js,devin_cloud.js,devin_proxy.js,media/*}` 后，**必须重新 vendor**
> 同步到 `core/dao-vsix/rtflow/`，否则二合一独立版跑的是旧代码。校验：
> `diff -q core/rt-flow/extension.js core/dao-vsix/rtflow/extension.js`。

### 公网多用户「道并行而不相悖」（会话隔离模型）

`/shell` 经 dao-bridge 暴露公网后，会有**多个浏览器同时连**。每个浏览器页 = 一个 `sid`
（`SHELL_HTTP_SHIM` 内 `sh_<rand>`），各自一条 SSE 通道（`_shellClients: sid→res`）。
隔离要点（`core/rt-flow/extension.js` 宿主侧）：

- **板块回推按 sid 隔离**：六大板块是**单一宿主、状态为号主共享**的 `_cloudProvider`。
  用户发起的 `cloudInit/cloudRelay/cloudReady` 经 `_shellCloudRun(sid, fn)` **串行化**执行，
  执行期间 `_shellCloudActiveSid` 锁定该 sid，宿主一切回推（含 `await` 后的异步回包）经
  `_shellCloudDispatch` **只发给该 sid** → 各用户各得其所、互不串台。任务间（无活跃 sid）
  的后台只读刷新（`refresh`）才广播给所有页（数据本为号主共享）。
  > 旧病灶（已修）：`cloudInit` 内 `setHostPost(()=>_shellBroadcast(cloudHost))` —— 把某用户
  > 的板块数据广播给所有连接的浏览器，公网多用户互相串台。源级护栏见 `test/unit.test.js`
  > 「/shell 多用户会话隔离」。
- 各浏览器页的标签集 / 已开的 Devin 对话 iframe **本就按页隔离**（每次加载独立）。

### ✅ 已解决：公网用户的「Devin 对话 / 多实例账号页」同源直达（旧「待完善」已落地）

旧病灶：`_shellResolveOpen` 曾返回 `http://localhost:<随机端口>/…`（绑 127.0.0.1·公网打不开）。
**现状（已归一）**：`_shellResolveOpen` 一律返回**同源相对 URL** `/sessions/<id>?dao_acct=<email>`
或 `/?dao_acct=<email>`，经**主端口 9920 同源反代**直出整 Devin SPA（`devinCloudProxyRoute`，
mode=`devin`）。每请求按 `dao_acct`（页面 query / 同源 XHR 的 Referer）**钉号注入该账号 auth1**，
多号并行各取各 auth 不串（见踩坑 7）。IDE 内（localhost:9920）与公网隧道（主口 9920）两端皆可达，
含真·上传框 → 拖拽桥可投递，与手机 APK 网页一致。

> 公网暴露：`/shell` 与 `/api/shell/*` 免 token（见 extension.ts needAuth 白名单），公网用户经
> dao-vsix 自带「公网穿透」板块（`bridgeStartTunnel` → cloudflared 快速隧道，绑 `ws.port` 9920）
> 即可打开同一张归一网页。注意：`addons/dao-bridge`（整机直连·机控 /api/*）是**另一条**隧道，
> 不代理 `/shell`；面向公网网页的隧道是 dao-vsix 本体的「公网穿透」板块。

### 道·公网渲染「穿透只传必要核心」（对照手机 APK 路线 · 三级缓存）

手机 APK 由 WebView `shouldInterceptRequest` 在原生层剥 CSP/XFO 并直取 app.devin.ai，故渲染
大头（JS/CSS bundle）走手机自身网络、隧道只传控制/鉴权。**桌面公网浏览器无此能力**：实测
app.devin.ai 静态资产由 S3 下发、**无 `Access-Control-Allow-Origin` 头**，浏览器 ES module
跨源 `import` 必被 CORS 拦死（旧 `/__web?u=app.devin.ai` 整页空白即此因）。302 跳真站亦受
module CORS 约束无解。故桌面端「公网渲染只传核心」的**正解 = 同源反代 + 三级缓存**：

1. **浏览器级**：不变资源（`/assets/*`）响应钉 `Cache-Control: public, max-age=31536000, immutable`
   → 每个公网浏览器首取后本地缓存，后续导航/多实例零穿隧。
2. **宿主磁盘 L2**（`~/.dao/asset-cache/`，键=`mode|path` 的 sha256）：内容哈希不变 → 落盘即永鲜；
   **跨宿主重载、跨多公网用户共享同一份资产** → 每个 bundle 全局只穿隧一次。见 `staticCachePut`
   / `staticCacheGetDisk`（extension.ts）。
3. **宿主内存 L1**：热点资产 Map（上限 256），命中最快。

> 经此，稳态隧道只承载 API/HTML 等**动态核心数据**（auth 注入后的对话流/列表），渲染负荷由公网
> 浏览器自身缓存承担 —— 即用户所述「网页渲染消耗公网浏览器的数据、穿透只传必要核心」的桌面实现。

---

## 三、内网穿透是去中心化的 —— 不需要 Cloudflare 账号、不存在必需的 Worker

- 默认通道 = **dao-relay**（`addons/dao-relay`）的零账号中继：`(session, token)` 配对，
  URL 形如 `…workers.dev/relay/<session>`，**零配置、无需任何账号**，插件启动即自动打通。
- **不要以为必须有 Cloudflare 账号 / 必须重发某个 Worker 才能穿透。** dao-bridge 本身是
  去中心化节点。**只有当用户自己想要「固定不变的公网域名」时**，才需要他自己登录
  Cloudflare（命名隧道）；那是可选项，不是前置条件。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dao-genesis/devin-remote](https://github.com/dao-genesis/devin-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
