---
trigger: always_on
description: 撰寫醫學文獻回顧（literature review）時使用。執行「三層外部知識管線 + 第二輪 PDF 深挖」流程：外部搜尋 → 全文落地 → 程式化比對，並在第一輪 PDF 閱讀完後追加第二輪參考文獻搜尋與下載。LLM 僅做改寫，所有事實必須能被 grep 驗回外部來源。觸發詞：「文獻回顧」「literature review」「整理 paper」「寫 review article」「醫學寫作」「PubMed / Europe PMC / OpenEvidence / Consensus 整合」。
---


# 醫學文獻回顧：三層外部知識管線 + 第二輪深挖

> **v2.0 變更摘要**（2026-04）：`claude.ai PubMed` connector 已退役，改用 `paper-search-mcp`（<https://github.com/openags/paper-search-mcp>）作為第一層搜尋與第二層下載的統一前端。新工具支援 21 個來源（arXiv / PubMed / bioRxiv / medRxiv / PMC / Europe PMC / Semantic Scholar / Crossref / OpenAlex / CORE / OpenAIRE / dblp / DOAJ / BASE / Zenodo / HAL / SSRN / IACR / CiteSeerX / Unpaywall / 可選 Sci-Hub），並內建 **`download_with_fallback`** 的 OA-first 下載鏈（source-native → OpenAIRE/CORE/Europe PMC/PMC → Unpaywall DOI → 可選 Sci-Hub）。

## 核心原則：**不信任 LLM 的內部知識**

LLM 對醫學細節（作者、期刊、年份、DOI、數字、樣本數）會產生**看似合理但錯誤的輸出**——這就是幻覺。臨床寫作一旦幻覺，代價是決策失誤。本流程從頭到尾守一條硬規則：

> **LLM 只做組織、改寫、語氣處理；所有事實性內容必須有外部來源可追溯，且能被 `grep` 驗回本地全文。**

LLM 不是知識來源，而是編輯。真相來自 paper-search-mcp 查到的 metadata、OpenEvidence / Consensus 的 citation、以及**本地 PDF→Markdown** 後可以 grep 的全文。

---

## 流程總覽

```
[1] 第一層：外部搜尋與真實性驗證（paper-search-mcp 多源並行）
        ↓
[2] 第二層：全文取得（download_with_fallback → LlamaParse → 本地 MD）
        ↓
[3] ★ 第二輪深挖：讀完 PDF → 抓內文 references → 重做第一層 + 第二層
        ↓
[4] 第三層：幻覺校正（DOI / metadata / 數字 grep）
        ↓
[5] 交稿：標記 📄（本地全文驗證）/ 📌（僅 abstract）
```

---

## 第一層｜來源搜尋與真實性驗證（不下載全文，先篩）

**主力工具：`paper-search-mcp`** — 一次呼叫覆蓋 21 個公開來源，內建去重。

### 1.1 Tier-A：並行多源搜尋（預設）

| 工具 | 角色 |
|---|---|
| **`mcp__paper-search__search_papers`** | Layer-1 高階工具：多源並行搜尋 + 去重；預設同時打 arXiv / PubMed / bioRxiv / medRxiv / Semantic Scholar / Crossref / OpenAlex / Europe PMC / PMC 等。**這是每個 topic 開工的第一刀**。 |

### 1.2 Tier-B：來源專屬 Search（針對性補強）

| 工具 | 用途 |
|---|---|
| **`search_pubmed`** | PubMed：最權威生醫索引；確認論文真實存在、作者/期刊/年份正確 |
| **`search_europepmc`** | Europe PMC：PubMed 的歐洲鏡像 + 更多 preprint / 灰色文獻 |
| **`search_pmc`** | PubMed Central：全文 OA 生醫論文 |
| **`search_semantic`** | Semantic Scholar：引用圖譜、influential citation 標記 |
| **`search_crossref`** | Crossref：DOI metadata 最權威來源 |
| **`search_openalex`** | OpenAlex：開放學術知識圖譜；institution / author ID 追蹤 |
| **`search_biorxiv`** / **`search_medrxiv`** | 生醫預印本，補 PubMed 尚未收錄的新研究 |
| **`search_google_scholar`** | bot-detection 活躍，需設 `PAPER_SEARCH_MCP_GOOGLE_SCHOLAR_PROXY_URL`；作為最後補網用 |
| **`search_unpaywall`** | DOI-centric OA metadata（**需設 `PAPER_SEARCH_MCP_UNPAYWALL_EMAIL`**） |
| **`search_core`** | CORE：全球 repository 聚合（建議設 `PAPER_SEARCH_MCP_CORE_API_KEY`） |
| **`search_openaire`** / **`search_dblp`** / **`search_doaj`** / **`search_zenodo`** / **`search_hal`** / **`search_ssrn`** / **`search_base`** / **`search_citeseerx`** | 依 topic 需要選擇性補網 |

### 1.3 臨床問答 + 立場統整（paper-search-mcp 不涵蓋）

| 工具 | 角色 |
|---|---|
| **`mcp__openevidence__oe_ask`** | 臨床問答專家系統，回傳必附 citation；快速建立觀點地圖 |
| **`mcp__claude_ai_Consensus__`** | 跨論文立場統整（贊成/反對某結論的研究比例） |
| **`mcp__tavily__tavily_search / tavily_research`** | 廣域 web 搜尋（guideline、學會聲明） |

**這層輸出**：一份「值得納入引用」的論文清單（含 PMID / DOI）。LLM 之後只能在這份清單內作文，**不得自創引用**。

### 1.4 論文真實性驗證（取代舊 `lookup_article_by_citation`）

收到 LLM 給的候選引用後（含 LLM 可能幻覺的情況），用以下順序驗：

1. 若有 **PMID** → `search_pubmed(query="<PMID>")` 或 `search_pubmed(query="<title>")` 比對 title/author/year
2. 若有 **DOI** → `search_crossref(query="<DOI>")` 取 metadata；或 `search_unpaywall(query="<DOI>")` 看 OA 狀態
3. 若都無 → `search_papers(query="<title> <first_author> <year>")` 打多源並行，看哪個來源回傳並比對 metadata
4. 三路皆無 → **判為 fabricated citation，剔除**

---

## 第二層｜全文取得（拿到 PDF → 轉 Markdown → 本地可 Grep）

### 2.1 主力：`download_with_fallback`（paper-search-mcp 內建 OA-first 鏈）

| 工具 | 行為 |
|---|---|
| **`mcp__paper-search__download_with_fallback`** | 按順序嘗試：**source-native download → OpenAIRE/CORE/Europe PMC/PMC 發現 → Unpaywall DOI 解析 → 可選 Sci-Hub**。一次呼叫自動走完 OA-first fallback chain。 |

這取代了舊版的 `research_hub → Europe PMC 直連 → scihub-mcp` 手動三段式。

### 2.2 來源專屬 download / read（某些來源內建 full-text 讀取）

| 工具 | 備註 |
|---|---|
| **`download_arxiv` / `read_arxiv_paper`** | arXiv OA 全文（`read_*` 直接回傳文字，可跳過 LlamaParse） |
| **`download_biorxiv` / `read_biorxiv_paper`** | bioRxiv |
| **`download_medrxiv` / `read_medrxiv_paper`** | medRxiv |
| **`download_semantic` / `read_semantic_paper`** | Semantic Scholar（只對 OA 有效）|
| **PMC / Europe PMC `download_*`** | OA PDF 下載 |
| **`download_iacr` / `read_iacr_paper`** | 密碼學預印本 |
| **`download_scihub`** | paper-search-mcp 內建 scihub fallback（與 `mcp__scihub__` 等價，使用者自負法律責任）|

### 2.3 備援：獨立 scihub-mcp（當 paper-search-mcp 的 scihub fallback 失敗時）

| 工具 | 備註 |
|---|---|
| **`mcp__scihub__find_pdf` / `download_pdf`** | Playwright bypass DDoS-Guard 的本地 scihub 實作；當 paper-search-mcp 的 `download_scihub` 因 mirror 故障失敗時的二備 |

### 2.4 付費牆論文

**使用者手動上傳 PDF** 到 `原始PDF/`，流程同下。

### 2.5 PDF → Markdown 轉檔

拿到 PDF 後用 **`mcp__llamaparse__parse_pdf_to_markdown`**（agentic tier）轉成 Markdown，存到 `原始PDF/*.md`。此時本地有「可程式全文檢索」的底本。

**驗收（關鍵）**：每個下載的 PDF 必須打開檢查內容語言 / 標題 / 作者是否與 metadata 一致——**防 sci-hub 映射錯誤（Polluted PDF）導致的資料污染型幻覺**。paper-search-mcp 的 OA-first chain 把 sci-hub 放在最後、降低發生率，但一旦觸發還是要肉眼驗。

---

## ★ 第二輪深挖（核心步驟）

> **第一輪只是入門票。第一批 PDF 內文的 References 才是真正的核心文獻來源。**

讀完第一輪 PDF 後執行：

### Step 1：從本地 MD 抽 references
- 用 Grep 搜尋每篇 MD 的 References / Bibliography 段落
- 對每篇 PDF 內容做摘要時，請 LLM **明確列出該文最關鍵的 5–10 個 cited references**（含 DOI / PMID / 作者年份）
- 特別關注：
  - 被多篇 review 共同引用的 seminal paper
  - 第一輪論文用來支撐核心論斷的 primary study

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zinojeng/medical-literature-review](https://github.com/zinojeng/medical-literature-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
