---
trigger: always_on
description: 开源独立插件（不与杰哥其他插件共享代码仓库），X榜单（xbangdan.com）出品。在 x.com 时间线/详情页每条推文互动栏末尾插「生成卡片」按钮，点击弹预览窗，把推文（头像＋昵称＋@handle＋日期＋正文＋配图＋互动数据）排成分享卡，支持白色/黑色/壁纸三种样式、下载 PNG / 复制到剪贴板；非中文推文可开谷歌翻译出双语卡（原文在上译文在下）。UI（预览 modal + popup）全部简体中文。
---

# SnapCard · X 推文卡片生成器

开源独立插件（不与杰哥其他插件共享代码仓库），X榜单（xbangdan.com）出品。在 x.com 时间线/详情页每条推文互动栏末尾插「生成卡片」按钮，点击弹预览窗，把推文（头像＋昵称＋@handle＋日期＋正文＋配图＋互动数据）排成分享卡，支持白色/黑色/壁纸三种样式、下载 PNG / 复制到剪贴板；非中文推文可开谷歌翻译出双语卡（原文在上译文在下）。UI（预览 modal + popup）全部简体中文。

## 架构（纯 MV3 插件，零后台服务）

- `manifest.json` MV3；host: x.com / twitter.com / pbs.twimg.com / translate.googleapis.com；`web_accessible_resources` 开放 `assets/*` 给 x.com/twitter.com（Wallpaper 默认背景图用）；`homepage_url` 指向 xbangdan.com
- `content.js` 注入按钮 + 抓 DOM 数据 + Shadow DOM 预览 modal（含 Style 选择器、自定义背景上传/重置），UI 文案中文
- `card.js` 卡片 DOM 模板：`buildCard(data, {theme})` 按 `theme` ("white"/"dark") 取一份 palette 对象出全部颜色，不写两份模板；另导出 `buildWallpaperFrame(cardEl, backgroundUrl)` 把白卡包一层背景图+阴影。日期中文格式（`formatTime` 输出「2026年8月19日 10:37」），显示在昵称行蓝V后面，不在底部
- `render.js` 卡片 DOM → SVG foreignObject → canvas 2x → PNG（零第三方依赖）；Wallpaper 模式直接把 `buildWallpaperFrame` 返回的整个 frame 节点丢进同一条渲染管线，背景图和卡片里所有 `<img>` 一视同仁走通用的 `inlineImages()` 转 dataURL，没有为背景图特殊处理
- `background.js` 图片代理下载转 dataURL（CORS 兜底）+ 谷歌翻译 translate_a/single
- `popup.html/js` 署名开关（chrome.storage.sync，默认关）+ GitHub 链接 + 底部 xbangdan 品牌栏（logo+文字，点击新标签打开 xbangdan.com）
- `assets/bg-aurora.jpg` Wallpaper 模式内置默认背景图（原创渐变，7 张里的第一张，2026-08-20 起替代苹果官方壁纸，见下方关键决策）
- `assets/xbangdan-logo.svg` popup 品牌栏用的 X榜单官方 logo（蓝紫渐变），**只用在插件自身界面，不进生成的卡片**

## 关键决策（2026-08-19 评估定稿，同日追加 Style 切换 + 中文化 + 品牌栏）

- 数据只读 DOM，不碰 GraphQL 接口，零风控
- 图片跨域：已验证 pbs.twimg.com 返回 `access-control-allow-origin`（回显 Origin），可直取；onerror 时走 background 代理兜底
- 长推文折叠：不自动展开，modal 里提示「先点开全文再生成」
- **署名默认关**，popup 可开（chrome.storage.sync key `watermark` 默认 `false`，content.js/popup.js 两处默认值必须保持一致）
- 翻译：谷歌免费接口，国内无代理会失败，失败时提示不报错崩溃
- 色盲安全：UI 不用红绿对，状态用文字＋明度；Style 选择器选中态用蓝底白字，未选中灰底
- **Style 三态**：White/Dark 是 `card.js` 里两份真实 palette；Wallpaper **不是**第三份 palette，固定用白卡，只是外面包一层背景图框（`buildWallpaperFrame`）——所以 `buildCard` 的 `theme` 参数实际只接受 "white"/"dark"，wallpaper 由调用方（content.js）自己决定「先建白卡→再套框」
- **Wallpaper 尺寸算法**：`buildWallpaperFrame` 要求传入的 `cardEl` 已经挂载在真实文档里（不能是离屏 detached 节点），用 `getBoundingClientRect()` 量出卡片实际宽高，各自加 15% 当 padding 得到外框尺寸；背景图用 `<img>`（不是 CSS background-image），这样 render.js 现成的「找所有 `<img>` 转 dataURL」逻辑不用改就能顺带处理背景图
- **主题记忆**：`chrome.storage.sync` key `theme`，默认 "white"，切换即写入，下次打开 modal 直接取上次选择
- **自定义背景**：`chrome.storage.local`（不是 sync，sync 单 key 8KB 放不下一张图）key `customBg`，上传时用 canvas 等比压到最长边 ≤2400px、JPEG q0.85 再存；有 customBg 时 Wallpaper 优先用它，没有则用内置 `assets/bg-sequoia.webp`（`chrome.runtime.getURL` 取）
- **UI 中文化**：预览 modal + popup 所有按钮/提示文案改简体中文（样式选择器 白色/黑色/壁纸，操作按钮 翻译/下载 PNG/复制图片/关闭/上传背景/恢复默认），中文标点全角；卡片本身的时间/互动数据格式不受影响（不在本次改动范围）
- **日期挪位**：卡片底部原来的日期行删掉，日期改中文格式挪到昵称行、蓝V徽章右边，13px 次要灰（跟随 palette.subtle，白卡 #536471／黑卡 #71767b，两个主题天然一致不用额外定义颜色）；昵称行改 `flex-wrap:wrap`，昵称过长时日期换到下一行而不是把卡片撑宽——之前昵称是 `nowrap+ellipsis` 截断，现在容器换行接管这个职责，索性把截断样式也去掉让昵称能完整显示
- **品牌植入原则（不可动摇）**：卡片本身（不管哪种样式、哪种导出方式）永远不出现 xbangdan 品牌，水印开关只控制「SnapCard」四个字；xbangdan 的 logo/文字/链接只出现在插件自己的 UI（popup 品牌栏 + manifest homepage_url + README），card.js/content.js/render.js/background.js 这四个「卡片生成链路」文件里不允许出现 "xbangdan" 字符串，改完都要 grep 一遍确认
- **Wallpaper 边距固定 60px（四边相等）**：`buildWallpaperFrame` 里 `WALLPAPER_PAD = 60`，不再按卡片宽高百分比算，避免竖长卡片上下边距明显大于左右
- **操作条按钮主次**：「复制图片」是主按钮（蓝底白字）排第一位，「下载 PNG」次按钮排第二位，「关闭」最后——大多数用户复制完直接粘贴发出去，比下载文件更高频
- **modal 超高内容处理**：面板本身 `maxHeight:90vh + overflow:auto` 兜底可滚动，操作条就是文档流里的普通子元素，滚动到底才看得到，不做 sticky 底栏（用户明确要求）；溢出时右下角浮一个「复制按钮在下面 ↓」提示胶囊，滚到接近底部（剩余 < 40px）自动淡出
- **主题记忆链路**：点样式按钮 → `saveStyle()` 立即写 `chrome.storage.sync` key `theme` → 下次 `handleGenerateClick` 里 `getSavedStyle()` 读回来初始化 `state.style` 和按钮选中态，读不到给默认 "white"，全链路已用 smoke.py 关模态框再重开验证过确实生效
- **隐藏互动数据**：`chrome.storage.sync` key `hideStats`，默认 `false`，跟 `theme`/`wallpaperBg` 同一套记忆模式；`buildCard(data, {hideStats})` 为 true 时整个 footer（含上边框分隔线）都不创建，不是简单 `display:none`，卡片直接以正文/配图收尾
- **隐藏时间**：完全照抄「隐藏互动数据」那一套——`chrome.storage.sync` key `hideTime`，默认 `false`；`buildCard(data, {hideTime})` 为 true 时昵称行蓝V后面那个 `dateSpan` 压根不创建（不是 display:none）；操作区 checkbox 排在「隐藏互动数据」后面，同一行同风格。两个 hide 开关目前是完全平行、互不影响的独立实现（各自独立的 storage key、独立的 state 字段、独立的 checkbox），没有抽公共封装——就两个开关，抽象反而增加阅读成本，等出现第三个类似需求再考虑要不要提取。
- **Wallpaper 背景选择**：一开始按用户要求做过 4 张纯 CSS/canvas 渐变生成的内置背景（`WALLPAPER_PAD` 那次之后的版本），当天晚些时候用户又改主意换成 7 张真实壁纸图（苹果 macOS 官方壁纸，全部方形 700-1002px webp），**canvas 渐变生成那套代码已整个删除**，改成走 `chrome.runtime.getURL(file)` 的统一 `BUILTIN_BACKGROUNDS` 数组（`{id, label, file}`），选中标识存 `storage.sync` key `wallpaperBg`（`custom:N` 或某个内置 id），自定义图升级成 `storage.local.customBgs` 数组（见下）。缩略图行只在 Wallpaper 模式显示，选中项 2px 蓝色描边，未选中 1px 灰色描边（`box-shadow` 模拟，避免描边把布局撑大）；「恢复默认」按钮已按用户要求去掉，点默认缩略图等效。**2026-08-20 这 7 张苹果官方壁纸已整批撤出，见下方 0.8.20 关键决策**，本条只保留架构层面的机制说明（`BUILTIN_BACKGROUNDS` 数组结构、`custom:N`、缩略图描边规则等）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jedeeai/snapcard](https://github.com/jedeeai/snapcard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
