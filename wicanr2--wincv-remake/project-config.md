---
trigger: always_on
description: 把 1999–2011 年的台灣共享軟體 **WinCV 0.52 (CView for Windows)** 逆向並以 Go + Ebiten
---

# WinCV Remake

把 1999–2011 年的台灣共享軟體 **WinCV 0.52 (CView for Windows)** 逆向並以 Go + Ebiten
乾淨重寫,目標 Linux / Windows / macOS / Android 四平台,介面做到與原版點陣像素對齊,
檔案系統等周邊功能做等價實作。

原作者 Lcc Wizard(林健總),原站 `cview.com.tw`。

## 授權與定位

remake 自己寫的原始碼、工具與文件走 **BSD 2-Clause**(見 `LICENSE`),repo 公開。

**原版 WinCV 的著作權屬於原作者**,LICENSE 不涵蓋它 —— 重寫是為了保存這份
文化資產,不是取得原版的權利。倚天字庫與原版隨附的字型、字典資料是第三方
版權物,**不打包進任何產物**。`internal/archive/ace` 有一段是從 BSD 授權的
droe/acefile 逐行移植的,其著作權聲明要一起帶。

**逐項的著作權與出處寫在 `NOTICE`。** 動到授權相關的敘述時改那一份,
不要在這裡複製一份狀態。

---

## 1. 目標軟體:已查證事實

下表每一列都是實測結果,不是推測。工具與重跑方式見 §8。

| 項目 | 事實 |
|---|---|
| `wincv052a.exe` | Inno Setup **5.3.8** 安裝檔(非程式本體)。PE32 GUI,8 sections,section 資料僅約 54 KB,其餘 5.8 MB 是 overlay |
| 解出內容 | 62 個檔,全部落在 `app/`。`innoextract` 可完整解開 |
| 版本 | WinCV **0.52**,`whatsnew.txt` 最後一筆 2011-11-24;`WINCV.IMG` 時間戳 2011-11-25 |
| `wincv.exe` (52 KB) | **Win32Forth v4 STC kernel loader**。7 sections,`.text` 僅 0x7FED bytes。import 只有 kernel32 / user32 / gdi32 / comdlg32 / advapi32 |
| `WINCV.IMG` (1.52 MB) | **應用程式本體**,Forth dictionary image。內含字串 `#Win32forth v4stc 0.1d by Lcc Wizard` |
| 執行方式 | `wincv.exe` 找不到內嵌 resource 就載入同名 `.IMG`(錯誤訊息 `FindResource: Failed to find the Win32Forth resource or .IMG file`) |
| 程式碼形態 | **subroutine-threaded code (STC)**,真 x86 指令,可反組譯 |
| word 數 | image 內 **3663** 個 code body,其中 **3633** 個帶標準 STC 序言 |
| 符號表 | **未被 strip**。header space 自 `0x122794` 起,解出 **9497 筆 header / 8957 個唯一名稱**,含 xt 位址;3509 筆 xt 直接命中 code body |
| 原版可執行性 | Wine 9.0 + Xvfb 可跑,視窗標題 `WinCV 0.52`,可截圖當 oracle(前提見 §7 雷 1) |
| 點陣字型 | NE FNT 2.0,全部定寬、涵蓋 0x00–0xFF:`cvga` **8×15**、`cvga1018` **10×18**、`cvga1224` **12×24**(`WinCV.fon` 與 `cvga.fon` 同規格)。ascent 11 / 16 / 20 |
| 字型註冊方式 | 程式以 `AddFontResource` 註冊自己的 `.FON`,再用 `CreateFontIndirect` 指名 face。Wine log 實測有 `Chosen: L"cvga Regular" (C:\wincv\wincv.fon)` |
| CJK 字形來源 | 不在隨附字型內。image 裡指名 **`新細明體`** → 原版全形中文由 Windows GDI 用系統字型繪製,字形隨使用者的 Windows 而異 |
| 全形格點 | 半形 8×15 → 全形 **16×15**。倚天 `STDFONT.15` 正好是 16×15、`ASCFONT.15` 正好是 8×15;`cvga1224` 12×24 對應倚天 `STD.24x` 24×24。同一個年代的規格 |
| 配色 | 語法設定檔用 **29 個具名顏色**,不是 16 色。名稱與順序取自 image 0x5692d 的斜線分隔清單(counted string,長度 227):black/dkgray/red/ltred/green/ltgreen/blue/ltblue/yellow/mildyellow/ltyellow/magenta/ltmagenta/cyan/ltcyan/gray/white/ltgray/purple/ltpurple/orange/ltorange/gooseyellow/bluegreen/inkgreen/mildwhite/mildgreen/mildcyan/mildmagenta。`keyword_*.cfg` 就是用這些名字指定語法上色。image 裡其實有 **46 個色彩 word**,那 29 個只是設定檔用得到的子集;檔案清單的副檔名配色用的是不在清單上的 `DIR-*` 系列。**RGB 值也在 image 裡**:每個顏色是一個 Forth word,body 有 0x24 個位元組,第 8-10 個就是 R、G、B(Win32 的 COLORREF 是 `0x00BBGGRR`,小端存放後記憶體順序正好是 R G B);word 的 xt 在標頭裡「名字結尾 +9」的那個 dword。抽取程式 `tools/palette.py`,結果在 `internal/render/raster.go` 的 `DefaultPalette` |
| Big5 字串表 | image 內的 UI 文字是 Forth counted string。加上「前一個 byte 等於長度」這道檢查後,15761 個候選收斂到 1293 個真字串;長度 ≥8 的有 845 筆,含選單、對話框、快捷鍵說明 |
| 附帶資料檔 | 英漢字典 `eng.txt` (5.5 MB) + `.dat`/`.idx`、`chi.txt.*`、KK 音標 `kk.txt.*`、`origin-verb.txt.*`、big5↔gbk/sjis/kor 對照表、`keyword_*.cfg` 語法上色、`ce.ful` 符號表、`default.fil` 書籤 |
| 解壓縮 | 原版外掛 Windows DLL:`unrar.dll` `unlha32.dll` `unarj32j.dll` `unacev2.dll` `tar32.dll` `CAB32.DLL` `7-zip32.dll` `bszip.dll` `aunzip32.dll` `libbz2` |
| 其他外掛 | `FreeImage.dll`(看圖)、`ijl15.dll`(Intel JPEG)、`cropdll.dll`、`md5.dll` |

功能面(取自 `file_id.diz`,原文為作者所寫,此處只列功能不照抄文案):
文字檔瀏覽、檔案與壓縮檔管理、看圖與縮圖列表、PE2 式區塊文字編輯器、HEX 編輯器、
Big5/GB/SJIS/KOR/Unicode 互轉(含檔名批次轉碼)、ANSI 彩色控制碼顯示、UNIX↔PC 換行轉換、
HTML/ANSI 碼去除、英漢字典與 KK 音標查詢、MD5 與 SFV(CRC32)檢驗。

---

## 2. 執行時架構

```
wincv.exe  (Win32Forth v4 STC kernel, 32 KB text)
   │  啟動 → 找不到內嵌 resource → 載入 WINCV.IMG
   ▼
WINCV.IMG  (1.52 MB)
   ├─ 0x000000            image header(10 個 dword,見下)
   ├─ 0x000000–0x12334c   code space:3663 個 STC word body
   ├─ 0x122794–0x186618   header space:9497 筆 word header(名稱 + xt)
   └─ 執行期再向上延伸(觀察到 xt 值超過 image 長度,如 0x1f3cc0)
   │
   ▼  Forth words 動態 LoadLibrary/GetProcAddress
Win32 API (user32/gdi32/…) + unrar.dll / FreeImage.dll / …
```

image header 實測值:

| offset | 值 | 判讀 |
|---|---|---|
| 0x00 | `0x00000000` | — |
| 0x04 | `0x019211d5` | 疑似 magic / checksum(**假設待驗**) |
| 0x08 | `0x0012334c` | code space 結束 |
| 0x0c | `0x00122794` | **header space 起點**(已用來成功走訪 9497 筆) |
| 0x10 / 0x14 | `0x00063e85` / `0x00063e84` | header space 大小 |
| 0x20 | `0x0040c158` | app base hint(**假設待驗**) |
| 0x24 | `0x00400000` | image base |

---

## 3. 逆向方法

### 3.1 STC 呼叫慣例(已由指令序列推導)

| 暫存器 | 角色 | 證據 |
|---|---|---|
| `ESP` | 資料堆疊(第二層以下) | `53` push ebx 存舊 TOS |
| `EBX` | **TOS cache**(堆疊頂端) | `53 bb 40 00 00 00` = push ebx; mov ebx,0x40 → 推入字面值 |
| `EBP` | **回傳堆疊指標**,向下成長 | 序言 `83 ed 04 8f 45 00` = sub ebp,4; pop [ebp] |
| `EDI` | 資料區基底 | `8b 9f 70 41 00 00` = mov ebx,[edi+0x4170];`89 9f 84 95 01 00` = mov [edi+0x19584],ebx。**指向 image base 或 user area 尚待確認** |

word 版面:

```
addr+0 : dword = addr+4          ← code field,指向自己的下一個位元組
addr+4 : 83 ED 04 8F 45 00       ← 序言(把 x86 call 壓的返回位址搬到回傳堆疊)
         …本體…
         83 C5 04 FF 65 FC       ← EXIT(add ebp,4; jmp [ebp-4])
```

「dword 值等於自身位址 +4」就是掃 word 邊界的判準,全 image 命中 3663 次。

### 3.2 符號表

`tools/forth_image.py` 已可完整走訪。header record 版面:

```
FF FF FF FF | 00 padding | name chars | count byte | dword seq | dword f2 | dword xt
```

`count` 在名稱**之後**(不是前綴),對齊靠前方 padding。`f2` 欄位語意未確認(**假設待驗**)。

產物:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wicanr2/wincv-remake](https://github.com/wicanr2/wincv-remake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
