---
trigger: always_on
description: 《Random Dice 2》互動式骰子樹攻略站。Astro 靜態站，部署在 Cloudflare Pages
---

# rd2-wiki

《Random Dice 2》互動式骰子樹攻略站。Astro 靜態站，部署在 Cloudflare Pages
（https://rd2-wiki.pages.dev/），GitHub `NatsuYukiowob/rd2-wiki`（public）。

## 這個專案的核心概念

資料正本是**兩個檔**（2026-08-22 起，#21 PR1）：

- **`data/dice-tree.svg`**——只有**幾何**（239 個 `<g class="node">`／248 條邊）：`data-id`、
  `transform`、形狀與 `stroke`、`<image>`、`data-wip`。**一個字都沒有**
- **`data/nodes.json`**——全部**文案**，以節點 id 為鍵：`name` `label` `type` `category?`
  `gameId` `cost` `maxLevel` `description` `awakening?`

外加 `data/icons/`（238 張 PNG，檔名 = 內容 sha256 前 12 碼）與 `data/tree-center.png`
（中央樞紐的合成圖，見下）。由社群發 PR 維護，CI 是唯一防線（維護者不可能逐行 review
SVG 的 diff）。

⚠️ **兩邊的 id 集合必須雙射**（規則 19）——這是「它們已經不同步」唯一會說話的地方。
join key 一律用 `data-id`，**不要拿座標配對**：浮點與 `transform` 一改就對不上。

為什麼要拆：`<title>` 是 `name` ＋ `description` 的完整副本（23.5 KB），規則 1 的存在理由
就是守那份副本；文案總計佔正本 48.6%（173,782 → 89,325 bytes）。拆完之後改一句描述＝
JSON 一行 diff，而不是一行 500 字元、rect/image/text 混在一起的 `<g>`。

⚠️ **`<text>` 標籤在 PR2（2026-08-22）也搬走了**，改成 `label` 欄位。它不是 `name` 的副本
（239 個裡 60 個是縮寫，`所有骰子傷害` → `全骰傷害`），是真資料——但留在 SVG 的話，線上
編輯器（#32）改一個縮寫仍然得對 SVG 動「行區塊外科手術」，而那是 #32 最脆弱的一段。
全搬之後編輯器任何文字操作都只碰 JSON。代價是正本用 Inkscape 打開是 239 個無名圖示，
補償是 **`npm run preview`**：把標籤與 id 注回幾何，產出 `data/dice-tree.preview.svg`
（gitignored）。那條動線是可逆的——**正本 → preview → normalize 逐位元組回到正本**，
`tests/tools/build-preview-svg.test.ts` 有一條測試守著它。

**外觀整個來自遊戲內的原圖**：2026-08-18 依遊戲內的骰子樹畫面 `RD2骰子樹 v1.0.1`
（`dice_tree_v1.0.1_fixed.svg`，素材不在版控內）重做，座標取原圖 ×0.5
（原圖節點座標全是 20 的倍數，減半後仍是 10 的倍數），圖示、配色、邊的粗細與顏色也都對過去。
那份原圖**只有畫面、沒有任何文字資料**（0 個 `<text>` / `<title>` / `data-name`），名稱／
花費／描述一律留在 `data/nodes.json`。

⚠️ **原圖的節點不是「一張圖」，是多層疊出來的**——底盤圖 ＋ SVG 漸層圖形 ＋ 帶 CSS filter 的
符號 ＋ 投影濾鏡。所以「換圖示」不能只是複製檔案，要跑 `npm run render-nodes`：它用真的
Chromium 把每個節點各自渲染成一張扁平 PNG（濾鏡與漸層由瀏覽器算，不重寫一份），再把結果
與尺寸寫回正本。這支**不掛在建置流程上**，一年跑不了幾次，CI 與貢獻者都不必裝瀏覽器。

核心功能：點一個節點 → 高亮它在 DAG 上的**所有祖先聯集**（去重、含自身、多重前置視為 AND）
→ 算出解鎖成本。

## 指令

```bash
npm run validate    # 資料驗證（規則 0–10、13–19，CI 守門員）
npm run typecheck   # tsc --noEmit（含 noUnusedLocals，會抓沒用到的 import）
npm run normalize   # 攤平 Inkscape 的圖層/matrix/相對路徑、清掉 <text> 與註解（送 PR 前必跑）
npm run preview     # 把標籤注回幾何，產出 data/dice-tree.preview.svg（不進版控，肉眼看版面用）
npm run add-icon    # 新增圖示，自動用內容雜湊命名
npm run render-nodes -- <遊戲原圖路徑>  # 用 Chromium 重畫全部節點圖示（遊戲改版才跑，見下）
npm run split -- <遊戲原圖路徑>         # 從原圖切出正本與圖示（重建整份資料時才用）
npm run build:data  # 產出 src/generated/tree.json + public/assets/
npm run build       # build:data + astro build
npm test            # 有 pretest 自動跑 build:data
npm run e2e         # 有 pree2e 自動跑 build
```

## 不變量（改動後務必重驗）

- 版本欄位有**三個、意義不同**：`data-game-version`（玩家看得到的遊戲版本，目前 1.0.3）、
  `<metadata>` 的 `resource bundle`（資料抄自哪一版遊戲資源包，目前 0.0.6）、
  `data-version`（正本自己的 schema 版本，目前 1.1.0）。首頁顯示前兩個，不要合併。
  ⚠️ `<metadata>` 開頭那句 `layout rebased on RD2骰子樹 v1.0.1 (2026-08-18)` 講的是**版面**
  抄自哪一版，跟 `data-game-version` 是兩件事——v1.0.3 沒有動幾何，所以它仍然停在 v1.0.1，
  不要順手一起改
- 節點 **239**、邊 **248**、根 5 個（`1001 2001 3001 4008 5002`）、多重前置 **14** 個
- **41 顆骰子各有一則 `awakening`**（7 骰點自動啟用的覺醒效果），其餘 198 個節點不准有這個鍵
  （規則 14）。覺醒**不是節點**：不用花錢解鎖、沒有前置、不進成本計算——所以它是骰子身上的
  一個欄位，不是第 240–280 個節點。做成節點會同時弄壞 239／248 與全樹解鎖成本
- **`gameId` 全 239 個都要有、且全檔唯一**（骰子 `D000`／符文 `D0000`／共通 `S0200`，
  規則 16）。它是正本與遊戲資料表唯一對得起來的鍵——⚠️ **刻意不進 tree.json**（站台不顯示，
  239 個字串要吃 0.55KB gzip），所以規則 16 是它唯一的防線，改壞了站台完全不受影響
- **`category` 只掛在 70 個玩家被動上**（`系別屬性 25／全骰屬性 15／系別技能 15／
  玩家被動 10／支援強化 5`，規則 16）。詳情面板有分類時顯示分類、沒有才顯示 type。
  `支援強化` 是本站的命名：遊戲資料表把支援角色與它的冷卻縮減都標成「支援」，照抄會寫出
  「支援 · 支援」與「玩家被動 · 支援」兩種都看不懂的組合
- `5201` 前置鏈 = **核心 42 ／ 金幣 23,000**（spec §6.4 基準）。
  ⚠️ 2026-08-21 從核心 66 降下來，因為鏈上的 `5002`／`5006`／`5008` 三顆骰子改成非成本解鎖
  （−24 核心）。舊文件與註解裡出現的「66」是那之前的基準，見下方規則 6(d) 那段的說明
- 全樹解鎖成本 = **核心 1,772 ／ 金幣 6,662,000**
- `dataIssue==='placeholder'` **0** 個、`no-growth` **0** 個
  （2026-08-21 依官方資料表補上那 5 個節點缺的 `(+每級增量)` 之後歸零，見規則 17）
- **`unlockVia !== 'cost'` 的節點共 9 個，全是骰子**：`default` 5（`1001 1005 1007 2001 3001`
  初始解鎖）、`quest` 1（`4008`）、`achievement` 3（`5002 5006 5008`）。
  來源是 `data/unlock-exceptions.json`，其中的 `note`（官方取得條件原文）會進 tree.json 的
  `unlockNote`，面板與 aria-label 優先顯示它而不是「任務解鎖」這種分類詞。
  ⚠️ **這不會改變 `meta.totalUnlockCost`**——那是「SVG 成本總和」（spec §2.1），刻意不排除
  非 cost 節點，站台一個地方都沒顯示它；會跟著變的是 `sumUnlockCost()` 的前置鏈計算
- ⚠️ **描述文字以「遊戲內實際顯示」為準，不是資源包裡的原始樣板。**
  資源包有沒填值的 `{n}` 佔位符時，遊戲並不是照樣印出來，而是**連同它所在的那一段一起不顯示**
  （2026-08-20 Yuki 逐個對照遊戲畫面）：

  | id | 資源包原文 | 正本（＝遊戲畫面） |
  |---|---|---|
  | 2403 | `攻擊速度增加5%(+{1}%)` | `攻擊速度增加5%` |
  | 5302 | `#僵硬範圍增加30%(+{1}%)` | `#僵硬範圍增加30%` |

  所以正本現在一個佔位符都沒有。下次拿新版資源包來對時這兩個會顯示成「跟上游不一致」，
  那是刻意的。

  ⚠️ **2026-08-21 更新：官方資料表補值的那幾個要照抄實值，不再砍整段。**
  資料表 v2 把 `5403`／`5307`／`5206` 三處原本沒填的數字補上了（`傷害增加30(最多100疊加)`、
  `30%機率額外獲得2疊加`、`#SP怪物的SP獲得量增加10(+5)`）。上游填得出來就用上游的，
  「連同那一段一起不顯示」只適用於**上游自己也沒有值**的情形——也就是上表剩下的兩個。

  ⚠️ **2026-08-22 v1.0.3 起 `5403` 的括號刻意不跟上游。** 資料表 v1.0.3 把它寫成全形
  `傷害增加30（最多100疊加）`，全站其他描述一律半形，正本維持 `(最多100疊加)`。
  下次拿新版資源包來對時這處會顯示成「跟上游不一致」，跟上面那兩個佔位符一樣是刻意的。
- ⚠️ **佔位符偵測機制留著，但真實資料已經沒有樣本了。** `parseGrowth` 的 `{n}` 判定、
  `dataIssue: 'placeholder'`、規則 9 的警告、面板的「數值待補」全都還在——上游隨時可能再
  冒出新的佔位符，那是唯一會提醒我們的東西。對應的測試因此**全部改成合成樣本**
  （注入一段 `{1}` 再驗），不再綁在某顆真實節點上；綁真實節點的話，資料一改測試就跟著消失，
  而那段程式還活著卻沒有任何東西守著。
  ⚠️ 2026-08-22 前注入夾具要**描述與 `<title>` 一起改**；文案搬進 `nodes.json` 之後只有一個位置
- 畫布 viewBox `0 0 2000 1700`；顯示尺寸**逐節點**寫在正本的 `<image width/height>`
  （骰子 50×53 ×41、符文 26×26 ×123、被動小 34×34 ×45、被動大 44×44 ×25、支援 51×47 ×5）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NatsuYukiowob/rd2-wiki](https://github.com/NatsuYukiowob/rd2-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
