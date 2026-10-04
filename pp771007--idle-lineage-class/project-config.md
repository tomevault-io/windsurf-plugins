---
trigger: always_on
description: | 查「有哪些外掛 / 哪些核心補丁」 | `docs/plugins.md` |
---

# 放置天堂（加掛版）— 專案規則

## 📇 細節在哪(需要時再讀,別憑印象動手)

| 要做的事 | 先讀 |
|---|---|
| 查「有哪些外掛 / 哪些核心補丁」 | `docs/plugins.md` |
| 改離線掛機(afk-offline) | `docs/offline.md` ＋ afk-offline.js 檔頭註解 |
| 玩家回報「離線結算跑很久」 | 跑 `node scripts/profile-offline.mjs --file <.testdata 檔> --slot N [--hot]`(拿真實存檔實測,別用新角色猜) |
| 想加速離線結算 | `docs/offline-batch-settle.md`(**草稿·尚未實作**;開頭補註說明為什麼優先順序被降低) |
| 查「手機耗電/發熱」熱點、做省電優化 | `docs/perf-battery.md`(熱點清單＋方案排序;第一批與「關閉光暈與濾鏡」已實作,其餘待議) |
| 做「存檔搬家 / 跨裝置轉移」 | `docs/save-transfer.md`(**評估·尚未實作**;整包 localStorage 的打包/還原做法、五種方案的優缺點) |
| 改 sw.js / 快取 / PWA | `docs/sw-pwa.md` |
| 改手機或平板版面、覆寫上游手機樣式 | `docs/mobile.md` |
| 同步上游 | 跑 `/sync-upstream`;背景與 CI 見 `docs/sync-upstream.md` |
| 改小百科 / 掉落查詢內容 | 跑 `/update-wiki`(內容準則與版面預算都在那支 skill 裡) |
| 準備 push | 跑 `/prepush`;發版跑 `/release` |

## 專案性質與架構（2026-07-19 起・純上游鏡像＋外掛層）

- 網頁放置遊戲。遊戲本體由原作者(巴哈姆特 秋玥)製作,原版:**https://shines871.github.io/idle-lineage-class/**;本站(加掛版):https://pp771007.github.io/idle-lineage-class/。
- **架構=「上游原版鏡像＋外掛層」**:核心(`js/NN-*.js`、`css/*`、`index.html`、`assets/`、`public/`)永遠是上游原文/原檔的位元組級鏡像;我們的所有功能都在**外掛層**——根目錄 `afk-*.js`(73 支)＋`sw.js`(PWA,上游沒有)＋極少量**錨點式核心補丁**(`scripts/apply-core-patches.mjs`)。
- 上游本機 clone:`D:/otherPersonRepos/idle-lineage-class`。**引用上游做任何判斷前先 `git -C <clone> fetch`**——舊 clone 會讓「上游也是這樣」的結論整個相反(踩過)。
- 同步狀態記在 `upstream-checkpoint.json`(`syncedUpstreamCommit`=目前鏡像的上游 commit)。
- ⚠️ **`assets/`、`public/` 下不可放我方獨有檔案**——CI 同步用 `rsync --delete` 鏡像,會被刪掉。外掛需要圖優先引用上游既有檔。

## ⭐ 修改原則(鐵則)

**🚨 絕不直接手改核心檔(`js/NN-*.js`/`css/*`/`index.html`)——下次同步上游會整包覆蓋,改了就丟。** 要動遊戲行為,依序考慮:

1. **外掛 monkey-patch(首選)**:核心函式都是全域,外掛包裝(`var _orig = fn; fn = function(){...}`)能解決絕大多數需求。afk-offline 連整套離線結算都是這樣掛的。
2. **錨點式核心補丁(最後手段)**:只有「外掛包不住」的才用——要抽函式、改函式簽名、改寫死的字面值、或改核心的熱點寫法。加進 `scripts/apply-core-patches.mjs`:靠「上游原文特徵字串」定位、冪等、**錨點找不到就 exit 1 大聲失敗**(不會默默壞)。補丁越少越好,現有 13 組(表在 `docs/plugins.md`)。
3. **index.html 不手改**:它=上游 index＋`scripts/afk-plugin-block.html` 注入到 `</body>` 前(sync 時自動重組)。**新增外掛 → 改 `afk-plugin-block.html`**(載入順序也在那裡管:afk-toggles 最先、afk-skin 最後),再把它的 `<script>` 行同步補進現行 index.html(或重跑 sync),有 DOM 掛點的加進 `scripts/smoke-hooks.mjs` 的 `need`。
4. **CSS 覆寫**寫在外掛注入的 `<style>` 裡(如 afk-mobile),不改 `css/*.css`。

> 此規則已有 hook 強制:PreToolUse(Edit/Write/MultiEdit)的 `.claude/hooks/core-file-guard.mjs` 會擋下對 `js/*.js`、`css/*.css`、`index.html`、`assets/`、`public/` 的直接編輯(腳本經 node 寫檔不受影響)。被擋到就改走外掛/補丁,不要繞過。

**外掛開關(afk-toggles.js,載入順序第一)**:每支外掛可被玩家單獨關掉——某支壞掉時玩家關掉它就能用原版繼續玩(逃生門)。契約:
- 純新增型外掛檔頭 `if (window.AFK_TOGGLES && !AFK_TOGGLES.enabled('<id>')) return;`;包核心函式型在 wrapper 內每次先問 `enabled()`,關掉就透明放行原函式。
- 載入時 `AFK_TOGGLES.register({id,name,desc,group,def})` 進開關面板。讀不到 AFK_TOGGLES 一律當開啟。afk-toggles 自己不可被關、不依賴任何外掛。
- **子選項**:`register({..., parent:'<父外掛id>'})` → 面板上縮排排在父項底下,且**父項關掉時 `enabled(子)` 一律回 false**(子項自己不必再問父項)。
  判準:**只有「父關掉＝子真的失效」才做父子**(資料源或入口依賴,如 offline→離線紀錄/選角掛機資訊、storage→設定選單裡的工具);
  「看起來相關」不算——手機那批(battlehud/mapbar/nozoom…)刻意各自判手機、不讀 `body.m-mobile`,掛成 mobile 的子項等於把下面那條死結耦合綁回去。

**🚨 不可停用的基礎設施,不能依賴「可被關掉的外掛」提供的東西**:逃生門(afk-toggles)、「讓開橫幅」這種所有裝置/所有外掛狀態都成立的需求,一旦去讀某支可關外掛設的變數或 class(`--orig-bar-h`、`body.m-mobile`),玩家把那支關掉就壞——最慘是連「把外掛開回來」的入口都被橫幅蓋住(橫幅 z-index 是 int 上限,壓得過任何外掛),形成無解死結。故橫幅讓位整組已抽成 `afk-banner.js`(無開關、載入序僅次 afk-toggles),afk-mobile 只留手機幾何專屬規則。兩條判準:
- **寫 `var(--某變數)` 或讀 `body.某class` 前先問「這誰設的?那支能不能被關?」** 能被關就要自己有保底(自己量一次/用同一組規則自己判)。
- **要寫進 afk-mobile 的規則,先問「桌機/平板關掉手機版面時還需不需要它?」** 需要就不屬於那支。

⚠️ 這類「A 外掛量測、B 外掛使用」的耦合在全開狀態永遠測得過 → smoke **第三輪**(手機+關掉 afk-mobile)驗逃生門可點、入口可見、`--orig-bar-h` 與 `#app-stage`/`#creation-screen` 仍讓開假橫幅;新增這類耦合時順手擴充該輪。

**外掛通用守則**:
- 優雅降級:需要的全域函式/元素不存在就 `console.warn` 後安靜停用,不可弄壞遊戲。
- **🚨 絕不可盲呼叫「會寫入玩家存檔」的原作函式**(踩過:主選單狀態呼叫 `saveGame()` 把玩家第 1 格蓋成 Lv.1 null、無備份可救)。要存檔資料**直接讀 `localStorage`**(`lineage_idle_save_<n>`);非寫不可時先驗 `player && player.cls`。任何會動玩家 localStorage 的操作,都要假設可能在「未載入角色/currentSlot 不是預期那格」被觸發。
- **🚨 外掛要存資料時,先問「這東西壞掉會不會影響遊戲」——答案決定能用哪個儲存**:
  - **會影響遊戲的(存檔、設定、任何遊戲行為讀得到的)→ 只能 `localStorage`**。**不少玩家是把 repo 下載下來直接開 `index.html` 玩(`file://`)**,而 `file://` 是 opaque origin、儲存政策各瀏覽器與版本都不同(本機 Chromium 實測 `file://` 下 IndexedDB 讀寫得了,**但這正是不能拿來賭的理由——一台可用不代表玩家那台可用**)。
  - **純診斷/取證、壞掉也不影響遊戲的 → 才可以放 IndexedDB**,而且**必須能在完全不可用時安靜停用**(open 包 try/catch＋`onerror` 一律轉成「這功能關掉」,絕不往上拋)。⚠️ 在效能敏感的裝置(iOS)上,**為了查問題而常駐的背景寫入本身就是新變因**(afk-blackbox 踩過,已整支移除)——量測工具要能一鍵拿掉,最好只在玩家主動開診斷時才跑。
  - 反過來也成立:**這類附屬資料不可以去佔 `localStorage`**——那 5MB 是存檔的地盤、本來就吃緊(`afk-quotawarn` 在盯 80% 門檻),多佔一點就可能害玩家存檔寫不進去。
  - 判準:**「file:// 打得開嗎」和「佔不佔存檔空間」兩條要一起過**;過不了就別存,改成算出來即用。
- 外掛插 DOM 錨「穩定容器 id」,不要錨父子關係——錨不到只會安靜消失,smoke 驗不到,改過首頁版面要人工掃。
- 覆寫「會被 `.hidden` 切換」的容器 display 時一律加 `:not(.hidden)`,否則畫面關不掉(踩過)。
- 覆寫上游「寫在 media query 裡」的樣式、或做手機/平板專屬元素 → 先讀 `docs/mobile.md`(兩套版面、同一條 MQ、橫幅讓位三條規則,都有 smoke 把關)。
- 外掛自建遊戲物件(如木人場 spawn 怪)欄位要對齊核心 `spawnMob`,缺欄位(如 `_born`)會整個系統安靜失效。
- 上游改版後外掛的「字串/DOM 結構假設」可能失效——同步後 smoke＋人工掃一輪首頁/手機版面。
- **🚨 搬運/備份/轉存類的東西,不可以「認得」被搬的內容**:正確性來自原樣進、原樣出。任何看得懂內容的判斷(key 名格式、欄位長度、「這是不是合法存檔」)都是對上游格式的假設,作者改個名字就把**所有合法檔案擋在門外**,而且是在玩家換裝置時才爆。要驗只能驗**自己寫進檔案的欄位**(format / schema / 項目數),不可以驗遊戲的東西。(afk-fullsave 踩過:`/^lineage_idle_save_\d+$/` 當「至少要有一個角色」的閘門。)

## 🚨 玩家看得到的字:一句都不能是廢話

判準只有一條:**這句話會讓玩家做出不同的動作嗎?** 不會就刪。

- 刪:容量、項目數、key 名、內部機制、設計理由(「刻意不挑是因為…」)——那些是註解該寫的,不是畫面該有的
- 刪:我們其實不知道的事(檔案會存到哪、能用什麼 App 傳)——寫了就是誤導(Android 下載資料夾還有 0 byte 的雷)
- 刪:在按鈕旁邊解釋這顆按鈕在幹嘛(按鈕名稱取好就夠)
- 留:他現在要做什麼決定、按下去會發生什麼、出事了該怎麼辦


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pp771007/idle-lineage-class](https://github.com/pp771007/idle-lineage-class) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
