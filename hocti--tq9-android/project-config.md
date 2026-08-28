---
trigger: always_on
description: 改嘢之前請先讀晒呢版。呢個 repo 係 Windows 版 `/mnt/d/dev/Q9/TQ9/`（C# WinForms）
---

# AGENTS.md — 九万輸入法 TQ9 (Android)

改嘢之前請先讀晒呢版。呢個 repo 係 Windows 版 `/mnt/d/dev/Q9/TQ9/`（C# WinForms）
移植過嚟嘅 Android system keyboard，行為要同原版夾得住。

---

## 一句講晒

九方過期專利 HK1035043 嘅 numpad 中文輸入法。撳 2~3 個碼查 `mapped_table` 出候選字，
字碼表／關聯字／同音字／繁簡表全部喺一個 sqlite 檔案入面，user 可以喺設定頁換走。

## 環境

| | |
| --- | --- |
| JDK | `/opt/android-studio/jbr`（要 `JAVA_HOME=/opt/android-studio/jbr ./gradlew …`） |
| SDK | `~/Android/Sdk`（`local.properties` 已寫死） |
| 版本 | minSdk 26 / targetSdk 36 / Kotlin 2.1 / AGP 8.13 / Gradle 8.14 |
| 模擬器 | AVD `Medium_Phone_API_36.1` |

```bash
JAVA_HOME=/opt/android-studio/jbr ./gradlew :app:assembleDebug :app:testDebugUnitTest
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell ime enable hk.tq9/.ime.TQ9InputMethodService
adb shell ime set    hk.tq9/.ime.TQ9InputMethodService
```

### 喺模擬器度試鍵盤，有三個陷阱

1. **一定要 `adb shell settings put secure show_ime_with_hard_keyboard 1`**。
   模擬器當自己有實體鍵盤，唔開呢個 setting 就唔會彈輸入法出嚟。
2. **唔好 `adb shell am force-stop hk.tq9`**。IME service 同 app 同一個 package，
   force-stop 會殺埋 IME，系統就會跌返去 Gboard。用 `am start` 就夠。
3. **每次 `adb install -r` 之後都要再 `ime enable` + `ime set` 一次**，
   系統會當佢 reinstall 而 reset。

設定頁最底本來有「試打」四個欄（普通／email／PIN／搜尋）同埋實時預覽，
**而家收埋咗**（`SettingsActivity.SHOW_DEBUG_SECTIONS = false`，user 唔想見到）。
`buildTryBox()` / `buildPreview()` 一行都冇刪 —— 想 debug 排位就改返做 `true`，
唔使開第三方 app。搜尋嗰欄係用嚟睇 `⏎` 有冇變 `🔍`
（`enterLabelFor()` 睇 `IME_ACTION_SEARCH`）。

---

## 唔好搞亂嘅嘢

### 九宮格排位係 numpad，唔係電話

`7 8 9` 喺最上、`1 2 3` 喺最落，跟返 `Q9Form.cs` 嘅 `ResizeAllButton()`。
底行係 `[0 佔兩格][取消]`；選字夠兩頁嗰陣兩格闊嗰粒 `0` 點變由設定話事
（見下面「選字揭頁」）。改過嚟電話排法就同原版打法唔同曬。

**右欄由上而下係 `☰／⇄`、`␣`、`⌫`、`⏎`**（2026-08-27 user 要求，`␣` 同 `⌫`
對調咗）—— 咁樣中文都跟返下面嗰條「`⏎` 上面嗰粒一定係 `⌫`」嘅規矩，
四款鍵盤一致。最上嗰粒平時係 `☰`，條 bar 常駐嗰陣變 `⇄`（見「工具列常駐」）。
左下角淨返粒 `🌐`（成格闊）—— 錄音搬咗上工具 bar，喺「貼上」隔籬，
亦都係左上角嗰粒鍵揀得嘅其中一個 `PadFunc`。

### 底行嘅規矩（英文／符號／純數字）

- **左下兩粒一定係「返去英文／中文」**（`Eng` 行先，跟住先至 `中`）—— 有兩個例外：
  中文九宮格自己就係中文，左下角淨係 `Eng`；**純數字頁**兩粒搬咗去**右上角**
  （user 要求，左下角讓咗俾 `0` `.` `-`）。
  英文嗰粒**寫 `Eng` 唔寫 `ABC`**（2026-08-25 user 要求，全部頁一致）。
  中文嗰粒**長撳 = 換輸入法**，做乜由設定頁揀（`Prefs.EngLongPress`）：
  `NEXT_IME` → `KeyAction.IME_SWITCH`（跳去下一個，跳唔到就跌落選單），
  `PICKER` → `KeyAction.IME_PICKER`（直接彈系統選單）。🌐 收埋咗，
  呢粒鍵係唯一入口，所以兩種做法都要有。
- **`⏎` 上面嗰粒一定係 `⌫`**。所以符號頁嘅分頁掣（`€£¥`／`?123`）同 `⌫`
  都喺倒數第二行嘅最左同最右，純數字頁嘅 `⌫` 亦都由右上角搬咗去 `⏎` 上面。
  第一頁嗰粒分頁掣**寫三個銀紙符號 `€£¥`**（第二頁頭一行就係啲銀紙），
  以前寫 `=\<`，冇人知係乜。
- 讓返出嚟嘅位：符號第一頁底行 space 右邊順住排 `, . ? ; /` 五粒（本來散喺
  上面兩行）；第二頁唔要標點，space 同 `⏎` 拉長，`numpad` 掣再升多一行。
- **純數字頁最左有一欄 `+ - * /`**（2026-08-25 加）：`-` 由底行搬咗上去，
  讓返出嚟嗰個位（`0` 右邊）擺咗粒 **`000`**（一次過打三個 0）。
  即係而家成頁係 5 欄，同中文九宮格一樣。
- **英文底行 space 右邊係 `, . /` 三粒**（本來係 `/` 喺 space 左、`?` 同 `.` 喺右）。
  `?` 已經冇咗獨立一粒 —— 佢係長撳 `/` 嘅第一個選擇（見下面）。

### 英文鍵盤排位（2026-08-24 大執過）

- **永遠有數字行**（`Prefs.FORCE_LATIN_NUM_ROW = true`）。設定頁嗰個開關收埋咗，
  但 `KEY_LATIN_NUM_ROW` 同「冇數字行就喺字母角落寫細字」嗰段 code 都冇刪。
- `asdfghjkl` **唔再靠拉長 `a` / `l` 收邊**：九粒一樣闊，兩頭各讓半格空位
  （`spacerKey(0.5f)`）。空位**唔會**入 `boxes`，所以撳落去會由 `boxNear()`
  snap 去隔籬真嗰粒鍵，唔會變死位。
- `,` 由 `zxcvbnm` 行搬咗落底行（頂咗本來個 `?`），讓返出嚟嘅位俾 `⇧` 同 `⌫` 拉長。
- **長撳字母大細階兩樣都揀得**：`ch()` 會按而家個 `ShiftState` 砌 variants ——
  排頭嗰個係粒鍵而家寫住嗰個（撳實唔郁放手 = 打返佢），第二個係另一個大細階。
  popup 揀返嚟嗰粒鍵帶 `Key.literal = true`，`typeChar()` 見到就**唔會**再套 shift
  （唔係特登揀個細階 `a` 會俾 shift 夾硬變返 `A`）。
- 標點三粒（`, . /`）長撳有 `PUNCT_VARIANTS`，左上角寫住細字提示。
  **三粒都唔跟「第一個 = 自己」規矩** —— 排頭嗰個係長撳一彈出嚟就已經停咗
  喺度嗰個（唔郁手指放開就出佢），粒鍵自己短撳攞得返：
  `,` → **Tab**（`\t`）、`.` → `;`、`/` → `?`。
- **Tab 冇字形**，畫出嚟一片空白，所以 `variantDisplay()`（`KeyDef.kt`）會換做
  `⇥` —— popup 同角落提示都要行呢個 helper，但係 `Key.variants` 入面存嘅、
  同埋最後 commit 出去嗰個一定要係真正嘅 `\t`。
- **變體多過螢幕裝得落就一齊迫窄**（`openVariantPopup`：
  `if (popupItemW * items.size > width) popupItemW = width / items.size`）。
  情願粒粒細啲都好過有幾個推咗出螢幕外面永遠揀唔到 —— `/` 有八個，
  用返 `max(鍵闊 × 1.1, 50dp)` 就一定爆。字太大 `KeyPopup` 自己會縮返。
- `?123` 長撳 = 直接跳純數字頁（`longAction = TO_NUMBER`），中文九宮格嗰粒一樣。

### 純數字頁：成頁唔准長撳

`NumberPadView.allowLongPress()` 一律回 `false`（`KeyboardBaseView` 嗰個 hook）。
打電話號碼／金額撳耐咗少少就彈個符號 popup 出嚟好煩，所以數字鍵**用 `num()`
唔用 `digitKey()`**（後者會帶 `variants`）。

闊度同貼邊**唔可以再自己計**：`RowsPadView.contentBounds()` 已經直接開一個
`PadMetrics`（一樣 5 欄 4 行）攞 `offsetX` / `contentW`，呢頁淨係要 override
`padGroup = PadGroup.CJK`，即係大細完全跟中文九宮格 —— 連工具 bar 左右拖出嚟
嗰個闊度倍數都跟。以前呢頁自己置中兼且封頂 360dp，中英切換嗰陣啲鍵會左右彈。

### `dataset.db` 唔會自動更新，舊機仲用緊裝機嗰陣嗰份

`Q9Db.ensureInstalled()` **淨係喺 `filesDir/dataset.db` 唔見咗嗰陣先由 assets 抄**——
user 可以喺設定頁換走個字碼表，夾硬覆蓋就會刪咗人哋自己揀嗰份。代價：
**由舊版升級上嚟嘅機，個 db 仲係當初裝機嗰份**。實測（2026-08-27，模擬器）
2026-08-21 嗰份 1.7MB 舊 db：

- `word_meta` **冇 `freq` / `code` 兩欄** → `topByCodePrefix` 查唔到（SQL 直接
  throw，佢自己 `runCatching` 食咗）
- `mapped_table` **冇 id `1010`** → 候選欄嘅預設字攞唔到

所以兩處都要有 fallback（`TQ9InputMethodService`）：`defaultPicks` 攞唔到 1010
就跌返落 1000（速選字表，即係以前嘅做法），`codePreview()` 空就跌返落
`defaultPicks`。**條 bar 吉住睇落似壞咗，情願出舊嗰套。** 想攞返新功能就叫 user
喺設定頁撳「還原內置字碼表」（會覆蓋佢自訂過嘅 db，所以唔可以靜靜雞自動做）。

加任何要新 schema 嘅嘢之前，記住問返自己：舊 db 會點？

### mapped_table 嘅 id 有特別意思

| id | 係乜 |
| --- | --- |
| `0` | 標點（首頁撳 0） |
| `1` | 開關標點成對（長撳 0） |
| `10`, `20`, … `90` | 姓氏表（撳咗第一碼之後再撳 0） |
| `10`~`999` | 正常字碼表，`weight` = 常用度 |
| `1000`~`1009` | 速選字表（⭐；首頁 = 1000，撳咗 1~9 之後 = 1001~1009） |
| `1010` | 候選欄嘅預設字（游標前面吉住／唔係中文嗰陣出，見「候選欄出乜」） |

打碼邏輯（`Q9Engine.press`）：夠三碼、或者中途撳 0 收尾，就查表出候選字。

### 字要用 grapheme cluster 拆

`Q9Db.splitGraphemes()` 用 `BreakIterator`，等同 C# 嘅 `StringInfo`。
用 `String.length` / `toCharArray` 會拆爛 emoji 同香港增補字符集。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hocti/tq9-android](https://github.com/Hocti/tq9-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
