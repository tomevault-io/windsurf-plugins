---
trigger: always_on
description: 一个助眠/放松网页 APP：选一段环境音（雨声、海边、夜虫、旷野等高质量白噪音），点开后**单段音频无限循环播放，且相邻两段有 5 秒重叠的交叉淡入淡出**，听起来无缝。目标是做成 PWA，手机点开、倒计时播放、熄屏持续。
---

# Sleep · 助眠环境音 — ambient 循环播放 PWA

## 这是什么

一个助眠/放松网页 APP：选一段环境音（雨声、海边、夜虫、旷野等高质量白噪音），点开后**单段音频无限循环播放，且相邻两段有 5 秒重叠的交叉淡入淡出**，听起来无缝。目标是做成 PWA，手机点开、倒计时播放、熄屏持续。

> **命名决策**：正式名定为 **Sleep**（副标题「助眠环境音」）。理由——助眠场景下，用户想睡觉时脑子里冒出的第一个词就是 sleep，零回忆成本，打开即用。早期代号 "Rain Loop" 是历史遗留（最初只有雨声），现已多音源，Rain 名不副实，故弃用。仓库名 `sleep`。
>
> **注意**：本地文件夹仍叫 `Rain`（未重命名——历史对话按文件夹索引，重命名会断链）。文件夹名与仓库名/产品名解耦，不影响。

## 语言/环境约定

- **语言**：中文。总结、Plan、WalkThrough、注释都用中文。
- **系统**：Windows 10，终端 PowerShell（命令分隔符 `;`）。本会话用 bash 风格。
- **委派子代理**：独立的搜索/实现/验证/多文件改动优先用 subagent，主会话编排审阅。
- **UI 主观体验验收由用户本人做**：过渡顺不顺、手感怪不怪、布局观感这类主观体验不派子代理用 Playwright 验收。子代理只做客观机械核对（DOM class、computed style、控制台报错、编译是否通过）。

## 项目结构

```
Rain/                              # 根目录即 Vite + Vue3 主项目（早期 app/ 子目录已平移上来）
├── CLAUDE.md                      # 本文件
├── README.md                      # 简介 + 音频/图标素材来源声明
├── index.html                     # Vite 入口（<title>Sleep · 助眠环境音</title>）
├── vite.config.js                 # dev server 配置：host:true 暴露局域网（手机测试），端口 5184
├── vite.config.pwa.js             # 生产构建配置：vite-plugin-pwa（manifest + service worker + workbox 缓存）
├── package.json                   # name: "sleep"，scripts: dev / build / build:pwa / icons / preview
├── .github/workflows/deploy.yml   # CI：push main → build:pwa → 部署 GitHub Pages（/Sleep/ 子路径）
├── scripts/
│   └── gen-icons.mjs              # 一次性：用 sharp 把 fa-bed 渲染成 icon-192/512.png（产物已提交，换图标再跑）
├── public/
│   ├── favicon.svg                # fa-bed 矢量图（Font Awesome Free，CC BY 4.0）
│   ├── icon-192.png / icon-512.png # PWA 安装图标（iOS「添加到主屏幕」只认位图）
│   └── audio-test.html            # 早期验证用独立 demo（双 audio 并发 + 自动循环），保留供回溯
└── src/
    ├── App.vue                    # UI 编排：选择页/播放页、音源下拉、面板开关（~170 行）
    ├── usePlayer.js               # 播放引擎 composable：双 audio 轮换、烤 WAV、倒计时、Media Session、缓存态
    ├── wav-encoder.js             # 16-bit PCM WAV 编码器（纯函数）+ 可运行自检
    ├── audio-sources.js           # 音源清单（key/name/file）——静态 import mp3，故无 node 自检
    ├── countdown.js               # 倒计时预设 + formatCountdown + 可运行自检
    ├── audio/                     # 6 段 Adobe 音源 mp3（192k）——经 bundler 内联/哈希，不在 public/
    │   ├── 01 - 打雷下雨.mp3
    │   ├── 02 - 倾盆大雨.mp3
    │   ├── 03 - 淅沥下雨.mp3
    │   ├── 04 - 海边礁石.mp3
    │   ├── 05 - 夜晚蟋蟀青蛙.mp3
    │   └── 06 - 旷野.mp3
    ├── main.js
    └── components/
        └── CustomDurationPicker.vue  # 自定义时长径向圆盘选择器（v-model:open + emit confirm ms）
```

> App.vue 早期是 ~700 行单文件 SFC（UI + 播放引擎 + 圆盘选择器全在一起），已按关注点拆为 UI 编排 + usePlayer composable + wav-encoder + CustomDurationPicker 组件。`docs/` 为早期设计稿存档，未参与构建。

dev server：`npm run dev`（用 `vite.config.js`），手机用输出的 `Network: http://<局域网IP>:<端口>/` 访问（手机与电脑同 WiFi）。
普通构建：`npm run build`（用 `vite.config.js` + `vite-plugin-singlefile`）→ 单个 `index.html`，mp3/woff2 字体全 base64 内联，拷到 U 盘 `file://` 直开可用（无网络、无 service worker）。
生产构建：`npm run build:pwa`（用 `vite.config.pwa.js`，注入 PWA）。两套配置分开——PWA 需要 service worker 与 manifest 这两个独立外链文件，dev/single-file 用不到，比 if/else 污染主配置清晰。部署走 `.github/workflows/deploy.yml`（push main 自动 build:pwa → GitHub Pages）。
**注意**：用户机器上常有多个残留 node dev 进程占用 5173~5184，Vite 会自动跳端口。清理由用户手动做，不要主动杀进程。

## 核心架构（已验收确定，勿轻改）

**淡变烤进 WAV + 双 `<audio>` 实例 + timeupdate 触发轮换。** 关键点：

1. **音频预处理（惰性、按音源 key 缓存）**：首次选中某音源时 `prepareOne(key)` —— fetch mp3 → `decodeAudioData` 解码成样本 → 对每个样本乘振幅系数（前 5s 淡入、后 5s 淡出）→ 手写 16-bit PCM WAV 编码器（[wav-encoder.js](src/wav-encoder.js)，Web Audio 无原生 encode）→ `Blob` + `URL.createObjectURL`，存入 `blobCache`（Map: key→blobUrl）。预处理与播放引擎均在 [usePlayer.js](src/usePlayer.js) composable 里。
   - 淡变**烤在波形里**，播放时**零 JS 控音量** → 精度无损（采样级）、熄屏无忧、无轮询粗糙问题。这是本方案能成立的关键。
   - **等功率淡变**：振幅系数用 `Math.sqrt(i/fadeSamples)`（非早期线性）。重叠段两路功率总和近似恒定，消除交叉处听感洼。淡入/淡出对称用同一 g。
2. **双 `<audio>` 实例**（A、B）都指向当前选中音源的 blob URL。`loop=false`（每段播一遍自然停）。`bindAudio(key)` 在切源时把两个实例的 `src` 换到新 blob。
3. **轮换触发用 `timeupdate`，不用 setInterval**：当前段挂一次性 `ontimeupdate`，当 `currentTime >= duration - 5` 时启动下一段。`duration` 自适应 → 换任何长度音频都自动取"结束前 5 秒"交接。
   - timeupdate ~250ms 粒度，但每段基于自身真实进度重算，**误差不累积**；环境音下偏差听不出。
   - 不用 setInterval 的原因：熄屏漂移 + 误差累积。
4. **同一时刻最多 2 段并发**：正在淡出的老段 + 正在淡入的新段，重叠 5 秒叠加，无缝。

## 多音源选择（已验收）

- 清单在 [src/audio-sources.js](src/audio-sources.js)：`AUDIO_SOURCES` 数组，每项 `{ key, name, file }`，其中 `file` 是 mp3 经 Vite 静态 import 解析出的 URL。**加音源只需往 `src/audio/` 丢 mp3 + 加一条 import + 这里加一行**。
  - **两套构建同源**：普通 build（`vite.config.js` + `vite-plugin-singlefile`）把 `assetsInlineLimit` 调高 → mp3 内联成 data URI，单 HTML 拷到 U 盘 `file://` 直开可用（fetch(dataURI) 在 file:// 放行，烤 WAV 路径成立）；PWA build（默认 limit）→ mp3 产物为 `/assets/<hash>.mp3`，运行时 CacheFirst 仍按 `endsWith('.mp3')` 命中。MP3 走 bundler 而非 public/ 是为让单 HTML 构建能内联它们。
  - **无 node 自检**：静态 import mp3 使 `node src/audio-sources.js` 跑不动（node 解析不了 mp3）。countdown.js / wav-encoder.js 的自检不受影响。
- idle 选择页有下拉 chip：选中 → `selectAudio(key)` 记忆到 localStorage（`rain:selected`）→ 惰性 `prepareOne`（下拉项显示"准备中…"）→ `bindAudio` 切换双实例 src。
- `blobCache` 跨音源复用：切回已烤过的音源是即时的，不重复解码。

## 倒计时（已验收）

- 预设 [src/countdown.js](src/countdown.js)：10 分 / 30 分 / 1 小时 / 2 小时 / 无限 / 自定义。
- **自定义 = 径向圆盘选择器**（独立组件 [src/components/CustomDurationPicker.vue](src/components/CustomDurationPicker.vue)，`v-model:open` + `emit('confirm', ms)`）：两段式（先选小时、释放后自动切分钟），SVG 圆盘 + Pointer Events，tap 与 drag 统一走 atan2→角度→值映射。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaujetZhao/Sleep](https://github.com/HaujetZhao/Sleep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
