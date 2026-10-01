---
trigger: always_on
description: MyR2D2 是**公開開源**的 Claude skillset repo（繁中本體、中英雙語觸發詞）。任何 session 在本目錄工作時，必須遵守以下約定。
---

# MyR2D2 — 開發約定

MyR2D2 是**公開開源**的 Claude skillset repo（繁中本體、中英雙語觸發詞）。任何 session 在本目錄工作時，必須遵守以下約定。

> 若本機存在 `.claude/local-rules.md`（不進 repo），開工前一併讀 —— 那裡放的是不適合公開的本機守門規則。

---

## 🔒 鐵則 1：這是公開 repo，動筆前先去識別化

repo 裡的 skill 與作者本機 `~/.claude/skills/` 的同名版本**是兩份不同的文件**，不是新舊版。公開版是把私人版泛化後的產物，抽換規則如下（每一條都有既成實例，照著做就不會漏）：

| 私人版寫法 | 公開版必須改成 |
|---|---|
| 真實人名（「當 Eric 說…」） | 一律「使用者」 |
| 私人專案／品牌代號、客戶名、人選姓名 | 刪除，或抽象成 `<專案>` 佔位符 |
| 私有 CLI／服務（自製 todo、Notion、Telegram、私人腳本路徑） | 換成零依賴的通用機制，並在文末補一節「進階：接上你自己的任務系統」 |
| 私人絕對路徑（`~/Documents/...`、`~/ClaudeProjects/<私人專案>`） | 刪除；範例路徑一律用 `~` 或 `<project>` 泛型佔位符 |
| 指向作者私人設定檔某節的交叉引用 | 改寫成自包含敘述，公開版不得依賴讀者看不到的檔案 |
| 寫死的模型型號（`Fable 5`／`Opus 5`／`opus`） | 檔位語彙：**旗艦檔／中檔（如 `sonnet`）／低檔（如 `haiku`）** |
| 帶日期的本機實測記錄（「2026-07-09 三連實測」） | 刪除日期戳，只留可長期成立的結論 |
| 跨機／iCloud／特定主機的環境細節 | 刪除，公開版假設單機通用環境 |
| 中文 skill 目錄名 | 英文 kebab-case 目錄名（`flight-to-calendar`、`save-all`） |

**送出前守門（commit 前跑，不是想到才跑）**：

```bash
git grep --untracked -inE "/Users/|~/(Documents|ClaudeProjects)/|Notion|telegram"
```

三個旗標／片段都是踩過坑才加的，別省：

- **`--untracked`** —— 少了它就只掃已追蹤檔案，剛寫好還沒 `git add` 的新 SKILL.md 會被跳過，而那正是外洩最可能發生的時機。它同時自動略過 `.gitignore` 排除的路徑。
- **`-i`** —— 少了它，官方拼法的 `Telegram`／`Notion` 全部漏網（私人版原文幾乎都是首字大寫）。
- **`~/(Documents|ClaudeProjects)/`** —— 只找 `/Users/` 抓不到寫成 `~/` 的私人路徑，而上表舉的例子本身就是 `~/` 記法。

⚠️ 這條指令是**篩子不是保險絲**：它掃的是 repo 檔案樹，抓得到的只有列進 pattern 的字面。跑過不等於乾淨，仍要人工複查。

**已知的預期命中**（不是外洩，別因此忽略其他命中）：

- 本檔上面那張表裡示範「私人路徑長什麼樣」的那一列 —— 用的是 `...`／`<私人專案>` 佔位符，不指涉任何真實專案。
- 作者署名 `Eric Lu (tingyulu)`、`github.com/tingyulu`（LICENSE、`plugin.json`、`marketplace.json`）與 repo 自身連結（README 安裝指令）。
- 第三方 MIT 致謝連結 `kieiken/ultracode-token-optimization`。

除這三類以外的命中，一律當成外洩處理到查清楚為止。本機另有完整的敏感詞清單，見 `.claude/local-rules.md`（該清單本身不進 repo）。

⚠️ **本檔（CLAUDE.md）也會被 commit 進公開 repo** —— 寫規範時同樣受本節約束，別把私人專案名寫進規則裡。

---

## 📐 鐵則 2：SKILL.md 的格式規範

### frontmatter

只有兩個欄位：`name`、`description`。**不加 `version`**（版號是整包的，見鐵則 4）。
唯一例外：衍生自第三方 MIT 專案的 skill 加 `license: MIT`（目前只有 `token-optimizer`）。

🔴 **`description` 的值一律用單引號包住** —— 這不是風格偏好，是 YAML 語法硬需求：

```yaml
description: '……時觸發。 English triggers: "a", "b".'
```

本 repo 的 description 必然含有 `English triggers: `（冒號＋空白）。在**未加引號的 plain scalar** 裡，`: ` 會被 YAML 當成 mapping 分隔符 → `mapping values are not allowed here` / `Psych::SyntaxError`，整支 skill 解析失敗。`pickup` 的 `status: pending` 也是同一個雷。

- 用**單引號**（值內部的 `"` 可原樣保留）；值裡若有 `'`（如 `don't`）改寫成 `''`。
- 別為了規避而把 `English triggers: ` 的冒號拿掉 —— 那個格式是鐵則 2 的一部分，該加引號的是整個值。

**改完必驗**（兩個獨立 parser，並且要有陽性對照確認 parser 真的在檢查）：

```bash
/usr/bin/python3 -c "
import yaml,glob,re
for f in sorted(glob.glob('skills/*/SKILL.md')):
    m=re.match(r'^---\n(.*?)\n---\n',open(f).read(),re.S)
    try: print('OK  ',f,list(yaml.safe_load(m.group(1)).keys()))
    except Exception as e: print('FAIL',f,type(e).__name__)
"
```

> 📌 **事故紀錄**：`v0.1.1`（2026-07-27 發布）的 5 支 SKILL.md **全部**是無效 YAML，`npx skills add` 0/5 成功——而且從發布日起公開 repo 一直是壞的，直到 2026-07-30 沙盒實測才發現。禍首正是 v0.1.1 引入的雙語觸發詞。肉眼看檔案、或確認「檔案存在、內容看起來對」都抓不到這種錯：**唯一有效的驗證是拿真的 YAML parser 去解**。

### description 的結構（順序固定）

```
<一句話定位> <具體做什麼／不做什麼>。當使用者說「A」「B」「C」時觸發。[補充句] ⚠️ <邊界提醒>。 English triggers: "a", "b", "c".
```

- 中文觸發詞用「」逐一包住、**相鄰排列不加逗號**，句尾接「時觸發。」（`flight-to-calendar` 用「時使用。」，語意需要時可換動詞）
- 英文觸發詞**固定收尾**，另起一句 `English triggers: `，雙引號包住、**逗號分隔**
- 中英**不是逐詞對譯** —— 中文是主體清單，英文是使用者實際會講的英文說法，各自獨立
- 觸發子句與 `English triggers:` 之間可插補充句，說明**產出**（`dropoff`：「產出＝…一張交接卡」）或**適用場合**（`pickup`：「也適合 session 開場主動跑一次」）
- ⚠️ 邊界提醒為選用，但凡是「做 X 但不做 Y」的 skill 都該寫（`save-all` 明寫「不重開機器」）

> 📌 **既有不一致，改到時順手收斂**：`token-optimizer` 的 description 在「」清單之外，另插了一段頓號分隔、沒有「」包住的「觸發詞：workflow、多代理、fan-out…」。新 skill 一律用「」包住，別複製這個寫法。

### 正文結構

1. `# <name> — <中文副標>`，`<name>` 逐字用 frontmatter 的 `name`（`/` 前綴用於斜線命令型 skill：`/save-all`、`/dropoff`、`/pickup`）
   - 既有例外：`token-optimizer` 的 H1 寫成 `# Token Optimizer`（Title Case）。要美化顯示名稱可以，**詞序不變**。
2. 一句話展開定位
3. `> 🤖 R2-D2 時刻：<Star Wars 類比>` —— 品牌彩蛋，放在說明之後、步驟之前。衍生作品改放 attribution blockquote（見 `token-optimizer`）
4. 主體：`## 為什麼需要` → `## 動作` / `## 步驟` → 規則濃縮
5. **必備一個規則濃縮區塊**（`## 鐵律` 或 `## 🚦 鐵則`）—— 把整份濃縮成幾條不可違反的規則，加粗關鍵詞＋emoji 前綴。位置有彈性，照現況三種都合法：
   - 收在最後（`save-all`）
   - 收在 `## 進階：接上你自己的任務系統` **之前**（`dropoff`、`pickup`）
   - 整支 skill 本質就是規則列表時，放在**開頭**（`flight-to-calendar`，文末是 `## 注意`）
   - 衍生自第三方、原文沒有這個區塊的，維持原狀不必硬加（`token-optimizer`，文末是 `## 8. 限制`）

### 行文慣例

- **標點**：敘事／文稿類內容用全形（，。、：「」（））；指令／log／程式導向的 skill（如日誌三支）可用半形。判準＝內容性質；單檔內部一致即可。既有檔不回溯改。反引號一律半形包指令
- 步驟**預設用 markdown 有序清單**（`1. 2. 3.`）掛在 `## 動作`／`## 步驟` 底下（`dropoff`、`pickup`、`flight-to-calendar` 都是）。只有步驟多到需要拆前置動作、或想讓每步能被單獨引用時，才升級成 H3 依序編號並從 `### 0.` 開始（目前有 `save-all`、`new-mission`）
- 子項目用圈碼 ①②③④⑤，後續步驟用同一組圈碼回頭對應，不重打項目名
- emoji 語意固定：⚠️ 風險／🚫 禁止／✅ 完成條件／🔁 重複性規則／📝 紀律／🔴 絕對規則／🔒 資料界線／安全／🤖 彩蛋
- **「驗證優先於宣告」是全 repo 的主題句**：凡是寫入動作，一律配上具體驗證指令（`wc -l`／`stat`／`grep`／`cat` 回讀）＋明講「別信工具回的『成功』字面」
- 零依賴優先：預設不綁任何外部服務；要接外部系統寫進「進階」節，並註明「檔案版是零依賴的最小公倍數，不是天花板」

---

## 🔁 鐵則 3：改一支 skill 的連動清單

skill 的行為／觸發詞／依賴一改，**同一個 commit 內**掃完下表。漏改會讓 README 描述與實際行為脫鉤：

🔑 **以「標題錨點」定位，不要只認行號** —— 行號會隨任何一次插入而整批推移（2026-07-30 加 skills.sh 安裝節就推移了四列）。下表行號是當日快照，對不上時以標題為準。

| 位置（錨點） | 行號快照 | 內容 |
|---|---|---|
| `README.md` 開頭定位句 | L11 | 「**12 支** skills」計數字串 |
| `README.md` skill 總表 | L13–26 | 一句話＋R2-D2 對應 |
| `README.md` `## 這套東西怎麼開發的` | L28–32 | 失憶引言＋自我修正敘事（二審缺陷數、測項數——測試計數一變這裡也要動） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tingyulu/MyR2D2](https://github.com/tingyulu/MyR2D2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
