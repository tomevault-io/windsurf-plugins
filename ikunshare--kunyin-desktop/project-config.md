---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

坤音（KunYin）桌面端：Electron + Vue 3 + TypeScript 的多音源聚合音乐播放器，是 Android 版坤音（Kotlin/Compose，`com.ikunshare.sound`）的桌面重写。聚合六大在线音源（`wy` 网易云 / `qq` / `qqc` / `kg` 酷狗 / `kw` 酷我 / `joox`），提供统一搜索、播放、歌单、下载、歌词、平台登录、LX 同步体验。代码纯 AI 生成。

构建链：electron-vite（开发/构建）+ electron-builder（打包），包管理器 npm。

## 常用命令

```bash
npm install          # 装依赖（postinstall 会 electron-builder install-app-deps 重建原生模块）
npm run dev          # 开发调试（electron-vite dev，热更）
npm run build        # 构建 main/preload/renderer 到 out/
npm run start        # 预览已构建产物（electron-vite preview）

npm run typecheck    # = typecheck:node（tsc）+ typecheck:web（vue-tsc）
npm run lint         # eslint --cache .
npm run format       # prettier --write .
npm test             # node --test tools/*.test.mjs（带 ts-resolve 钩子，可直接 import 仓库 .ts）
npm run build:wasm   # 重编 native/qmc-wasm 并回写 src/main/crypto/qmcWasmBinary.ts（需 Rust）

npm run build:unpack # 构建 + 解包目录（不产安装包）
npm run build:win    # 构建 + Windows 安装包
npm run build:mac    # macOS
npm run build:linux  # Linux
```

- 单跑一侧类型检查用 `npm run typecheck:node` / `typecheck:web`。
- 构建产物在 `out/`，打包产物在 `dist/`。electron-builder 的瘦身 `files` 排除规则在 `electron-builder.yml`，**必须全部留在顶层**，win/mac/linux 段内不能再写 `files` 字段（否则顶层整份被静默忽略，见文件内注释）。

## 三层结构与共享层

Electron 三进程，源码分三棵 + 一个共享层：

- **src/main**（主进程）：唯一触网、触盘、跑加密/解密、SQLite 的地方。含 providers、crypto、store、cache、net、auth、modules、ipc、audio、windows。
- **src/preload**：contextBridge 暴露 `window.api`（类型化 `WindowApi`）、`window.electron`（@electron-toolkit/preload）、`__INITIAL_APPEARANCE__`。渲染层**不直接触网**，一切经 `window.api.<域>.<方法>` 转发到主进程。
- **src/renderer/src**（渲染层）：Vue 3 + Pinia + vue-router，两个入口 `index.html`（主窗口）+ `desktop-lyrics.html`（桌面歌词悬浮窗）。
- **src/common**：三端共享的类型/常量/纯函数，统一从 `@common` 导入（出口是 `src/common/index.ts`）。

路径别名（`electron.vite.config.ts` 与两个 tsconfig 的 `paths` 同步维护）：

- `@common` → `src/common`（三端通用）
- `@renderer` → `src/renderer/src`（仅渲染层）
- `music-lyric-kit` → `src/renderer/src/lyric/kit/main`、`music-lyric-player` → `src/renderer/src/lyric/player/main`（vendored 歌词引擎，业务侧 import 名不变）

## 核心数据模型（@common）

- `MusicItem`：按 `type`（`MusicSource`）分发的**可辨识联合**。公共字段在 `BaseMusicItem`，各源有专属字段（qq 有 `mid`，kg 有 `hash` 等）。`local` 只出现在歌单里、不走 Provider/后端。
- 唯一键 `getMusicItemKey(item)`：一般 `type_id`，酷狗 `kg_hash`（无 hash 的歌词重定向目标用 `kg_lyric_<downloadId>`）。列表去重、歌词缓存都靠它。
- 音质 `QualityId`（`128k|320k|flac|hires|master|atmos|atmos_plus`）：**唯一排序真源是 `QUALITY_IDS`**（低→高，`src/common/constants.ts`），徽标/降级/取流/下载顺序全从它派生。AI 音质的屏蔽与降级见 `blockedQualityIds` / `qualityFallbackOrder` / `qualityUpgradeOrder`。
- `Lyric`：主进程产出的原始歌词容器（`lrc/trans/roma/char/chroma/phonetic`），渲染层再交给 vendored 歌词引擎解析。

## Provider 架构（音源）

`src/main/providers/` 是音源抽象层：

- `base.ts` 的 `BaseProvider` 抽象类定义全接口（search / 专辑歌手 / 歌单 / MV / getLyric / resolveMediaInfo / 评论等），各源在 `providers/<source>/` 实现，默认方法返回空结果。
- `index.ts` 的 `registry` 注册六大源，`getProvider(source)` 按源取实例；登录态由 `src/main/auth/credentials.ts` 启动时读 cookie 注入 `provider.credentials`。

**移植铁律**：音源实现是 Android Kotlin/C++ 的**逐字移植**，禁止凭记忆或网络试错（签名/加密/设备指纹风控极敏感）。改某源前先读 Android 版坤音（<https://github.com/ikunshare/kunyin>）对应 Kotlin 源。非显然坑已散落在 `src/main/providers/` 与 `src/main/crypto/` 的注释里，例如：QQ 搜索必须走签名 `musics.fcg`（不是 `musicu.fcg`）；QRC 是改版 DES（`crypto/qqDes.ts`）；移植 C 位运算加密务必逐处 `>>>0` 保证 uint32 回绕（否则 mflac 解密输出=输入）。

## 播放地址解析与音频流（关键链路）

播放/下载共用的统一解析入口是 `src/main/providers/getUrl.ts` 的 `resolveMediaInfo(item, qualityId)`：

1. 本地歌曲直接短路；
2. 命中 `urlCache` 直接返回；
3. Provider 自定义 `resolveMediaInfo` 优先，否则回退自建后端 `POST https://c.wwwweb.top/app/getUrl`（body `{platform, musicId, quality, authst}`；`platform` 用 `BACKEND_PLATFORM` 映射，`musicId` 按源取 mid/hash/id，`authst` 来自卡密）；
4. 结果按直链时效缓存（切回听过的歌不重复请求）。

返回 `{url, ekey}`：**无 ekey = 明文直链**，渲染层 `<audio>` 直连（CSP `media-src` 放行 http/https）；**有 ekey = 加密流**（QQ mflac 等），走自定义协议 `kunyin://`（`src/main/audio/protocol.ts`，`protocol.handle` 边下边解密、透传 Range、支持 seek）。卡密校验在 `src/main/auth/manager.ts`（`POST /app/checkAuth`，校验通过即把卡密当 `authst` 用）。

### 取流连接的生命周期（socket 泄漏 =「放久了每首歌都超时」）

`kunyin://` 每一路取流都握着一个 Chromium socket，而**每主机只有 6 个、全进程 256 个**。漏掉的连接不会自己回来；攒满之后新的取流连额度都申请不到（请求在连接池里静默排队），而我们的「等响应头」计时器照样在走——表现就是**「放久了之后任何歌曲都 upstream header timeout，重启应用才恢复」**。下面三条都是实测结论，动这条链路前务必先读：

- **`protocol.handle` 的 `request.signal` 从不触发**（Electron 44 实测：换 `<audio>.src`、seek、渲染层 `AbortController` 全试过，一次都不 abort）。能观测到下游放弃的唯一信号是「我们返回的 `Response.body` 被 cancel」——即便 handler 是事后才返回的，Electron 也会补这一刀。所以取消靠 `bindToRequest` 里 pipeTo 落地反推；等响应头那段还没有 body 可 cancel，由 `openSlot` 的「同 token、且一个字节都没吐过的旧请求直接顶替」兜住。
- **`resp.body.cancel()` 不归还 socket**，只有 abort 那次 fetch 的 `AbortController` 才会（三种收尾方式对照实测：什么都不做 / body.cancel / abort，只有最后一种让活连接归零）。所以上游连接按**租约**管理（`UpstreamLease`，控制器**活过响应头阶段**），`net/request.ts` 的 `drainResponse` 同样是先 abort 再 cancel。
- **异步生成器取消不掉卡在 `await` 里的自己**：`AsyncGenerator.return()` 排在未决的 `next()` 之后，`fetchAndStore` 卡在 `await reader.read()`（上游半开）时 `finally` 永远跑不到，那条连接就永久泄漏。所以 `streamFrom` 的 cancel 必须**先 abort 再** `return()`，且所有上游流都要穿过 `audio/streamGuard.ts` 的空闲看门狗（`UPSTREAM_IDLE_TIMEOUT`，20s 一个字节都没有就掐断）。看门狗用 `highWaterMark: 0` 是**语义不是调优**：HWM=1 会让流自己预读一块，于是暂停播放时也始终挂着一次 pull，健康连接会被误判成停摆。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ikunshare/kunyin-desktop](https://github.com/ikunshare/kunyin-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
