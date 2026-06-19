---
trigger: always_on
description: >
---


# PPTX Presentation Generator

將任何文件或報告轉換為可直接用於公司報告的 PPTX 簡報。

---

## 推薦使用流程

**使用者不需要手寫投影片資料。** 推薦流程如下：

```
使用者提供需求描述 / 原始內容
        ↓
AI 根據「提示詞樣板」自動產出 slides.json
        ↓
pptx-generate 生成 PPTX 簡報
```

### 提示詞樣板 (Prompt Template)

本工具提供一份標準化的提示詞樣板：**[pptx_generator/assets/prompt-template.md](pptx_generator/assets/prompt-template.md)**

此樣板包含：
- 完整的 slides.json schema
- 所有 11 種投影片類型的範例
- 內容量限制規則
- 排版規則
- 完整的輸出範例

**使用方式：**
1. 將 `prompt-template.md` 的內容貼給任何 AI（ChatGPT、Claude、Gemini 等）
2. 接著提供你的需求描述或原始內容（文件、報告、筆記、口頭指示皆可）
3. AI 產出 `slides.json`
4. 執行 `pptx-generate --input slides.json --out output.pptx -v`

**在 AI IDE 中更簡單：** 直接說「幫我做簡報」，AI 會自動讀取此樣板並完成全部流程。

---

## 使用者輸入

### 必要

| 輸入 | 說明 |
|------|------|
| **內容來源** | 報告、文件、需求描述、口頭指示、URL — 任何形式皆可 |

### 可選

使用者可以用自然語言提供以下任何資訊，AI 自動理解並調整：

| 輸入 | 範例 | 未提供時的行為 |
|------|------|---------------|
| 模板檔案 | 「套用 ./template.pptx」 | 優先用 `pptx_generator/assets/default-template.pptx`；不存在則用空白簡報 |
| 目標受眾 | 「給老闆看」「給 IT 團隊」 | 依內容性質自動判斷 |
| 語言 | 「用英文」「中英混合」 | 跟隨內容來源語言；混合時預設繁體中文 |
| 頁數 | 「控制在 15 頁」 | 依內容量自動估算 |
| 風格 | 「圖表為主」「精簡摘要」 | 依內容性質平衡 |
| 輸出路徑 | 「存到 docs/report.pptx」 | `output_presentation.pptx` |
| 品牌色 | 「用 #007A33」 | 有模板繼承模板；無模板用 `#2B579A` |
| 字體 | 「用 Noto Sans TC」 | 中文 `微軟正黑體`，程式碼 `Consolas` |
| 既有投影片資料 | 「用這份 slides.json / .yaml / .md」 | 從 Phase 0 開始完整規劃 |

**使用者不需要知道這些參數。** 直接說「幫我做簡報」就能跑。

---

## 支援的輸入格式

工具支援三種輸入格式。推薦讓 AI 根據 prompt template 產出 JSON，但也可手動撰寫：

### 1. JSON（推薦 — AI 產出 + 完整控制）

由 AI 根據 [prompt-template.md](pptx_generator/assets/prompt-template.md) 自動產出，或手動撰寫。

- 範例：[pptx_generator/assets/example-slides.json](pptx_generator/assets/example-slides.json)

### 2. Markdown（手動撰寫最直覺）

用 Markdown 撰寫投影片內容，工具自動轉換為簡報結構：

```markdown
---
title: 專案進度報告
version: 2026-Q2
---

# 專案進度報告
Q2 2026 Review

## 大綱
- 專案背景
- 系統架構

## 關鍵成果
- 查詢延遲下降 42%
  - p95 由 2.3s → 1.3s
```

Markdown 對應規則：
- `# H1` → 封面（第一個）或章節分隔（後續）
- `## H2` → 新投影片
- 項目符號 → bullet_points
- 程式碼區塊 → code_demo
- `> 引用` → speaker notes

> 注意：Markdown 格式僅支援 `title_slide`、`section_slide`、`bullet_points`、`code_demo`。
> 需要 `kpi_slide`、`table`、`two_column` 等進階類型時，請使用 JSON 或 YAML。

### 3. YAML（可讀性佳）

比 JSON 更易讀寫，支援扁平格式（不需要巢狀 `content`）：

```yaml
title: 專案進度報告
version: 2026-Q2

slides:
  - type: title_slide
    title: 專案進度報告
    sub_title: Q2 2026 Review

  - type: bullet_points
    title: 關鍵成果
    points:
      - 查詢延遲下降 42%
      - 錯誤率下降至 0.4%
```

> YAML 輸入需要安裝 PyYAML：`pip install pyyaml`

---

## Pipeline

```
Phase 0  規劃大綱（AI 根據 prompt-template.md 自動執行）
Phase 1  內容解析 → slides.json（AI 產出，或使用者提供 JSON / YAML / Markdown）
Phase 2  Mermaid 圖表渲染
Phase 3  python-pptx 組裝
Phase 4  品質驗證
```

- **Phase 0 + 1 由 AI 自動完成**：AI 讀取 [pptx_generator/assets/prompt-template.md](pptx_generator/assets/prompt-template.md) 中的 schema 與規則，根據使用者提供的內容自動產出 `slides.json`。
- 若使用者已提供投影片資料檔（JSON / YAML / Markdown），跳過 Phase 0 和 Phase 1，直接進入 Phase 2。
- 範例檔案：[pptx_generator/assets/example-slides.json](pptx_generator/assets/example-slides.json) / [pptx_generator/assets/example-slides.yaml](pptx_generator/assets/example-slides.yaml) / [pptx_generator/assets/example-slides.md](pptx_generator/assets/example-slides.md)
- 提示詞樣板：[pptx_generator/assets/prompt-template.md](pptx_generator/assets/prompt-template.md)
- 依賴：`python-pptx`、`requests`、`Pillow`（YAML 輸入另需 `pyyaml`）

### Quickstart

**Windows (PowerShell):**
```powershell
pptx-generate --input slides.md --out output.pptx -v
```

**Linux / macOS:**
```bash
pptx-generate --input slides.md --out output.pptx -v
```

完整參數範例：

**Windows (PowerShell):**
```powershell
pptx-generate --input slides.yaml --template assets/default-template.pptx --out output.pptx --brand-color "#007A33" --font "Noto Sans TC" --footer "Company · Confidential" --version-label "2026-Q2 v1.2" --watermark "DRAFT" --page-numbers -v
```

**Linux / macOS:**
```bash
pptx-generate \
    --input slides.yaml \
    --template assets/default-template.pptx \
    --out output.pptx \
    --brand-color "#007A33" --font "Noto Sans TC" \
    --footer "Company · Confidential" --version-label "2026-Q2 v1.2" \
    --watermark "DRAFT" --page-numbers \
    -v
```

> `--json` 仍可使用（向後相容），但建議改用 `--input`。

未指定 `--template` 時，優先使用 `pptx_generator/assets/default-template.pptx`，不存在則用空白簡報。
未指定 `--footer` / `--version-label` 時，會自動使用 `presentation_metadata.title` / `.version`。

---

## Phase 0 — 大綱規劃

> **此階段由 AI 自動執行。** AI 讀取 [pptx_generator/assets/prompt-template.md](pptx_generator/assets/prompt-template.md) 中的規則來規劃大綱。

1. 根據內容量與需求，估算合理頁數。
2. 列出每一頁的標題與 slide type。
3. **大綱固定為第 2 頁**（封面之後），使用 `outline_slide`。
4. **每頁只講一個主題**。內容太多時拆成多頁（如「需求清單 1/2」「需求清單 2/2」）。
5. 大綱項目超過 10 個時，考慮合併章節或分兩頁大綱。

---

## Phase 1 — 內容解析 → slides.json

> **此階段由 AI 自動執行。** AI 根據 [pptx_generator/assets/prompt-template.md](pptx_generator/assets/prompt-template.md) 中的 schema、slide types、內容量限制，將使用者的原始內容轉換為 `slides.json`。

### slides.json Schema

```json
{
  "presentation_metadata": {
    "title": "string",
    "version": "YYYY-MM-DD"
  },
  "slides": [
    {
      "id": 1,
      "type": "<slide type>",
      "content": {
        "title":        "string — 所有類型，≤ 20 中文字（40 英文字元）",
        "sub_title":    "string — title_slide / section_slide",
        "points":       ["string array — bullet_points / outline_slide"],
        "mermaid_code": "string — architecture_diagram",
        "description":  "string — architecture_diagram",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paul0728/pptx-generator](https://github.com/paul0728/pptx-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
