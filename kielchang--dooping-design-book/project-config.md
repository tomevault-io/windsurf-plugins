---
trigger: always_on
description: > 本檔網路正本：<https://kielchang.github.io/dooping-design-book/AGENTS.md>（與 GitHub repo 根同步；
---

# 取用指南（給其他專案）

> 本檔網路正本：<https://kielchang.github.io/dooping-design-book/AGENTS.md>（與 GitHub repo 根同步；
> 若你讀到的是 `/preview/` 底下的副本，那是開發中的版本，取用一律以正式站為準）。
> 機器地圖：<https://kielchang.github.io/dooping-design-book/llms.txt>

這個 repo 是**設計方向的正本**。其他專案不在自己的 repo 裡重新發明按鈕、表格與確認流程，
而是從這裡取用。這份文件是「怎麼取用、什麼不能改」的一頁式契約；
完整說明在文件站 <https://kielchang.github.io/dooping-design-book/>。

給 AI agent：接手一個要遵照本設計語言的專案時，先讀完這頁，再讀文件站對應章節。
**不要憑印象重寫元件**——元件已經存在，用下面的指令裝進來。

## 三層，相依強度刻意遞減

| 層 | 內容 | 取用方式 | 改動權 |
| --- | --- | --- | --- |
| `packages/tokens` | 設計 token（語意色、間距、字級、陰影、動態） | `npm install @dooping/tokens` | **不可改語意，只可改值** |
| `packages/react` | React 參考實作（registry 項目全清單見 [`/r/index.json`](https://kielchang.github.io/dooping-design-book/r/index.json)） | `npx shadcn add <URL>` | 複製後就是你的，隨便改 |
| `book/docs`（模式與頁面章） | 操作模式（問題→做法→取捨→反例）＋五種頁型的組成規範 | 讀懂，用你的技術棧實作 | 不含程式碼 |

理由見 [ADR-0004](https://kielchang.github.io/dooping-design-book/adr/registry-over-npm-package/)（元件一定會被改，所以不發套件）
與 [ADR-0005](https://kielchang.github.io/dooping-design-book/adr/tokens-are-the-only-hard-dependency/)（token 幾乎不會被改，所以它才是契約）。

## 取元件：shadcn registry

```bash
npx shadcn@latest add https://kielchang.github.io/dooping-design-book/r/data-table.json
```

相依會自動一起裝——`data-table` 會帶上 `table` / `input` / `button` / `select` / `tooltip` / `utils`。
全部可用項目列在 <https://kielchang.github.io/dooping-design-book/r/index.json>，
單品 URL 一律是 `/r/<name>.json`。

### 有哪些元件、每種頁型裝哪些

- **清單正本＝`/r/index.json`**：每項含 `name`／`title`／`description`／`url`，機器直接讀。
  一行列舉：

  ```bash
  curl -s https://kielchang.github.io/dooping-design-book/r/index.json | jq -r '.items[] | "\(.name)\t\(.title)"'
  ```

- **人讀版**＝文件站[元件總覽](https://kielchang.github.io/dooping-design-book/components/overview/)。
- **要組整頁的話別逐個挑**：五種頁型的文件末尾各有一條「最小安裝集」指令，一次裝齊——
  [清單頁](https://kielchang.github.io/dooping-design-book/pages/list-page/)、
  [明細頁](https://kielchang.github.io/dooping-design-book/pages/detail-page/)、
  [表單頁](https://kielchang.github.io/dooping-design-book/pages/form-page/)、
  [儀表板](https://kielchang.github.io/dooping-design-book/pages/dashboard/)、
  [設定頁](https://kielchang.github.io/dooping-design-book/pages/settings-page/)。

**大相依提醒**：`graph-canvas` 會自動帶進 `@xyflow/react`（本 registry 唯一的大型外部相依，
由邊界守衛隔離在單一檔案）。裝之前先確認你要的不是零相依的 `charts`（八種 SVG 圖）。

**前置條件**：專案要有 `components.json` 與 `@/*` 路徑別名。沒有的話先 `npx shadcn@latest init`。
Tailwind 的 `content` 掃描範圍要涵蓋落點（`./src/**/*.{ts,tsx}` 已含 `components/dooping/`）。

**落點是固定的**，不要改：

```
src/
├── components/dooping/    ← 元件（.tsx）
└── lib/dooping/           ← 工具（utils、use-sort、csv、download、forms-diff）
```

放在 `dooping/` 子目錄是為了讓「哪些是設計中心來的」一眼可辨，
之後上游修 bug 時你才找得到要同步哪幾個檔案。

### 宿主前置條件：樣式基座（preflight）

元件的 utility class 只宣告 border-width；「`border-style: solid`、`border-width: 0`、
預設邊框色」由 Tailwind preflight 提供。**宿主沒有這層基座時元件不會報錯，
只會安靜地變形**：邊框整批消失（只有寬度沒有樣式）、裸按鈕露出瀏覽器原生
灰底凸框、表格吃到宿主的格線。看到這三種症狀，先查基座，不是查元件。

- **標準 Tailwind／shadcn 專案**：`shadcn init` 標配 `@tailwind base`，天然滿足。
  建議再加一條（shadcn 慣例，把「不帶色的 border」接到 token）：

  ```css
  @layer base {
    * { border-color: hsl(var(--border)); }
  }
  ```

- **把元件嵌進有自己 CSS 的既有站台**（後台框架、文件站、CMS——關掉 preflight
  的宿主）：不要全站開 preflight（會打爆站台既有樣式），改在元件所在的 scope 內
  鋪等價基座——本 repo 的文件站就是這種宿主，作法照抄
  [`book/src/css/demo-base.css`](https://github.com/kielchang/dooping-design-book/blob/main/book/src/css/demo-base.css)。
  注意 **portal 內容**（Dialog／Select／Tooltip／資料表篩選面板）掛在 `body` 直下，
  逃出容器子樹，scope 必須一併涵蓋。取捨與驗收方式見
  [ADR-0010](https://kielchang.github.io/dooping-design-book/adr/demo-host-baseline-contract/)。

## 取 token

```bash
npm install @dooping/tokens
```

這是**唯一建議的硬相依**。四個進入點，挑你的宿主吃得下的用：

```css title="純 CSS（任何宿主）"
@import "@dooping/tokens/tokens.css";

.my-alert {
  background: hsl(var(--danger) / 0.1);
  border: 1px solid hsl(var(--danger) / 0.35);
  color: hsl(var(--danger));
}
```

```js title="tailwind.config.js"
module.exports = {
  presets: [require("@dooping/tokens/tailwind-preset")],
  content: ["./src/**/*.{ts,tsx}"],
};
```

```ts title="JS API（Canvas 圖表、伺服器端 PDF、Figma plugin…）"
import { semanticColors, chartColors, TOKENS_VERSION } from "@dooping/tokens";

chartColors("dark");   // 8 色色盲友善色票
semanticColors();      // 35 個語意色（HSL 三元組）
```

第四個是 `@dooping/tokens/tokens.json`（來源正本，給非 JS 工具鏈讀）。

`tokens.css` 是純 CSS 變數、不含任何 Tailwind 指令，所以不用 Tailwind 也能用。
深色模式 `.dark` class 與 `[data-theme="dark"]` 屬性兩種鉤子都內建，切換就一行：

```js
document.documentElement.classList.toggle("dark");
// 或走屬性：document.documentElement.setAttribute("data-theme", "dark")
```

要改 token 值請改 `packages/tokens/src/tokens.json`，**不要手改 `dist/` 或 `src/tokens.data.ts`**——那是產物。

## 不可改的契約

複製走的元件原始碼是你的，隨便改。但下面這幾條一改，跨專案的一致性就沒了：

1. **語意色的名稱與意義。** `--danger` 就是危險、`--success` 就是良好。
   換品牌色請改 token 的**值**，不要改名字，也不要拿 `--warning` 去表示別的東西。
2. **琥珀色是「已改動未送出」的保留色**，不作他用。見 [ADR-0002](https://kielchang.github.io/dooping-design-book/adr/amber-reserved-for-dirty-state/)。
3. **深色模式鉤子**掛在 `document.documentElement`，`.dark` class 與 `[data-theme="dark"]` 屬性擇一即可（兩種都內建支援）。
   掛在 wrapper 上會讓 Dialog / Select / Tooltip 這類 portal 浮層抓不到。
4. **不要靠顏色單獨傳達語意。** 狀態要同時有文字或圖示——見[無障礙原則](https://kielchang.github.io/dooping-design-book/accessibility/principles/)。

## 相容性與版本

**以 `main` 為參照。** `dev` 是開發中的分支，不要拿它當來源。

兩層的版本模型不同，因為相依模型不同：

| 層 | 怎麼鎖 | 怎麼知道自己落後了 |
| --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kielchang/dooping-design-book](https://github.com/kielchang/dooping-design-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
