---
trigger: always_on
description: - 使用 `source ~/.zshrc` 來載入環境變數
---

# QChoice AI Collaboration Guidelines for SQL教學文件專案 (Claude Version)

- 使用 `source ~/.zshrc` 來載入環境變數
- 使用 `conda activate fju` 來啟動 Python 環境

## 專案概述 (Project Overview)
本專案為 SQL 與 AI 大數據情境處理教學文件專案，旨在提供完整的 SQL 教學內容與 AI 整合示範。Claude 在此專案中擔任深度分析、複雜邏輯設計與程式碼審查的角色。

## 架構 (Architecture)
- **文件結構**：採用模組化設計，區分基礎教學、進階應用與實戰案例
- **配置管理**：使用 YAML 格式配置檔案，確保設定的可讀性與維護性
- **安全性**：教學範例中不包含真實敏感資料，使用環境變數管理所有配置
- **整合焦點**：聚焦於 SQL 與 AI 技術的整合應用，模組化設計便於擴展

## 開發工作流程 (Development Workflow)

### **核心原則：規格驅動開發 (Specification-Driven Development)**

你的首要任務是遵循一個嚴謹的、由規格驅動的開發流程。所有程式碼的修改、測試的建立，以及文件的更新，都必須以 `docs/SPEC.feature` 和 `api-spec/openapi.yaml` 這兩個規格文件為唯一的真相來源 (Single Source of Truth)。

### **主要工作流程 (Primary Workflow)**

每一次的調整請求都必須嚴格遵循以下五個階段的順序。在開始任何階段前，執行以下前置檢查。

#### **前置步驟：檢查交接事項 (Pre-check Handover)**

1. **檢查檔案**：每次開始工作前，應檢查是否有 `worklog/PROGRESS.md` 檔案。
2. **參考交接**：如果檔案存在，則參考其內容，包括目標需求、已完成階段的總結、目前卡住的問題點、下一步行動建議，以及任何必要的上下文資訊，然後接續開發。
3. **清理檔案**：若交接事項已完全處理，則刪除或標記該檔案為已完成，以避免重複處理。

#### **階段一：規格定義與更新 (Specification)**

1. **分析需求**：根據使用者的請求，判斷是新功能還是既有功能的調整。
2. **更新規格文件**：
    *   **功能規格 (`docs/SPEC.feature`)**：更新或新增 Gherkin 格式的功能描述，確保其清晰地定義了使用者場景與預期行為。
    *   **API 規格 (`api-spec/openapi.yaml`)**：同步更新或新增 API 的端點 (endpoints)、請求/回應模型 (schemas)、參數等，確保與 `SPEC.feature` 的描述完全一致。

#### **階段二：程式碼實作與調整 (Implementation)**

1. **實作程式碼**：根據階段一更新後的規格文件，調整應用程式的程式碼，以實現所需功能。
2. **參考既有慣例**：若專案中存在 `.github/copilot-instructions.md`, `CLAUDE.md`, `GEMINI.md`, `AGENTS.md` 檔案，應優先參考其內容，以了解專案的特定慣例與架構決策。

#### **階段三：測試與驗證 (Testing)**

1. **建立或修改單元測試**：
    *   嚴格依照 `docs/SPEC.feature` 及 `api-spec/openapi.yaml` 的規格撰寫單元測試。
    *   確保所有功能，包含成功路徑與邊界條件，都被測試案例完整涵蓋。
2. **執行測試**：**在每次修改完程式碼後，都必須立即執行所有單元測試。**
3. **迭代修正**：如果測試未通過，**必須回到階段二**，繼續調整程式碼，直到所有單元測試都成功通過為止。**在測試完全通過前，不得進入下一個階段。**

#### **階段四：文件產出與更新 (Documentation)**

在確認程式碼與測試都符合規格後，對以下文件進行全面評估與更新：

1. **AI 協作指令 (`.github/copilot-instructions.md`, `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`)**：
    *   **評估**：檢查協作指令的**準確性**及**完整性**，確保其反映了最新的程式碼架構與 API 規格。
    *   **優化**：若有不足，則進行優化或產生新內容。專注於描述專案的宏觀架構、關鍵開發流程、以及 `api-spec/openapi.yaml` 中定義的 API 整合點。
2. **專案說明 (`README.md`)**：同步更新專案說明文件，確保使用者能理解最新的功能與 API 使用方式。
3. **安全政策 (`docs/SECURITY.md`)**：
    *   檢查 `docs/SECURITY.md` 是否存在。
    *   若不存在，則建立該檔案並包含以下內容。若已存在,請確保內容符合最新標準。

    ```markdown
    # 安全政策 (Security Policy)

    ## 範圍 (Scope)
    [請定義此政策涵蓋的專案範圍，例如：API 服務、前端應用程式等]

    ## 資料分級 (Data Classification)
    [請定義資料的敏感度等級，例如：公開、內部、機密]

    ## 威脅情境 (Threats)
    [請列舉主要的威脅情境，例如：未經授權的存取、資料外洩、服務中斷]

    ## 最低安全基線 (Minimum Security Baseline)
    [請定義開發時應遵循的最低安全要求]

    ## 漏洞通報 (Vulnerability Disclosure)
    我們非常重視並感謝所有來自外部研究員的安全回報。
    - **回報管道**：請優先使用 GitHub 的 **Security Advisories** 功能私下回報（若您有權限），或寄信至 **green07111@noreply.github.com**。
    - **我們的承諾**：
        - 我們承諾在 **72 小時內**回覆，確認已收到您的通報。
        - 我們承諾在 **7–30 天**內提供修補或緩解計畫（依據漏洞嚴重度）。
        - 我們採行「協調式揭露」（Coordinated Disclosure）；在修補完成後，我們將與您協調，共同決定公開漏洞細節的時機與方式，並在公告中向您致謝。
    - **快速指引**：為了方便研究員，我們建議在網站根目錄布署 `/.well-known/security.txt` 檔案，以指引通報流程（依據 RFC 9116）。

    ## 法規與準則
    - **法規遵循**：我們的服務設計遵循台灣《個人資料保護法（PDPA）》與其相關子法，確保僅蒐集業務所需資料、履行告知義務，並保障資料主體的權利。
    - **開發基準**：我們的研發流程參考 **OWASP Application Security Verification Standard (ASVS)** 與 **OWASP API/REST Cheat Sheets** 作為安全開發與程式碼撰寫的基準。

    ## 回報格式建議
    為了讓我們能更快地重現並處理問題，您的回報請盡可能包含以下資訊：
    - **影響範圍**：受影響的端點、功能或元件。
    - **重現步驟（POC）**：提供詳細的重現步驟，包含任何必要的設定、程式碼片段或請求範例。
    - **建議修補方向**：若您有任何修補建議，請一併提供。
    - **是否涉及個資**：說明此漏洞是否可能導致個人資料外洩。
    - **是否可被自動化濫用**：評估此漏洞被大規模利用的可能性。

    **提醒**：若您的回報涉及機敏資料，請使用加密附件，並在寄信時先向我們索取公鑰。
    ```

#### **階段五：工作記錄 (Work Log)**

在完成以上所有任務後，將本次對話與調整的過程總結至工作記錄檔案中。

1. **檔案路徑**：`worklog/REFACTOR_SUMMARY_{YYYYMMDD}T{HH}.md`，其中 `YYYYMMDD` 為當前日期（例如：`20251015`），`HH`為24小時制當前時刻（例如：`09`）。
2. **內容**：簡潔地總結本次修改的範圍，包含規格、程式碼、測試與文件的變動。
3. **操作**：如果該日期的檔案已存在，則在檔案末端追加內容；如果不存在，則建立新檔案。
4. **排除規則**：此工作記錄不應包含對 `README.md`、`.github/copilot-instructions.md`, `CLAUDE.md`, `GEMINI.md`, `AGENTS.md` 這些文件的修改摘要。
5. **無法一次完成工作時的交接**：如無法一次完成工作，需將目標需求以及目前階段性總結報告更新或創建寫入 `worklog/PROGRESS.md` 內交接，以便後續 Agents 可以繼續目前工作。內容應包含：原始需求描述、已完成階段的總結、目前卡住的問題點、下一步行動建議，以及任何必要的上下文資訊。

### **工作流程總結**

嚴格遵循 5 步驟規格驅動流程：
1. 規格更新（`docs/SPEC.feature` + `api-spec/openapi.yaml`）
2. 實作
3. 單元測試
4. 文件更新（`README.md`、`.github/copilot-instructions.md`, `CLAUDE.md`, `GEMINI.md`, `AGENTS.md` 等）
5. 工作記錄（`worklog/REFACTOR_SUMMARY_YYYYMMDDTHH.md`）

若無規格變更，從步驟 1 開始；測試失敗則回滾步驟 2。強調「規格先行」、測試必過與文件同步，適用所有變更。

## 命令與輔助工具 (Essential Commands & Helper Tools)

### **開發命令**
```bash
# 啟動 Jupyter Notebook
jupyter notebook

# 執行 Python 測試
python -m pytest tests/

# 檢查程式碼品質
pylint *.py

# 格式化程式碼
black *.py
```

### **測試命令**
```bash
# 執行所有測試
python -m pytest

# 執行特定測試檔案
python -m pytest tests/test_sql.py

# 產生測試涵蓋率報告
pytest --cov=. --cov-report=html
```

### **配置設定**
- 使用 YAML 作為配置檔案格式
- 配置檔案位置：`config/settings.yaml`
- 環境變數優先於配置檔案

### **輔助工具：Claude 的專長領域**

作為 Claude，你在以下領域具有特殊優勢：

1. **深度程式碼分析**：能夠深入理解複雜的程式邏輯，識別潛在的效能瓶頸與安全漏洞
2. **架構設計**：擅長設計可擴展、可維護的系統架構
3. **程式碼審查**：提供詳細的程式碼審查意見，包含最佳實踐建議
4. **複雜問題解決**：能夠處理需要多步驟推理的複雜技術問題

### **與其他 AI 工具協作**

- **Gemini CLI** (`gemini -p "..."`): 適合快速資訊搜尋與簡單查詢
  - 範例：`gemini -p "找出專案裡所有使用 SQL 的地方"`
  - 範例：`gemini -p 'google_web_search(query="Python SQLAlchemy best practices")'`

- **GitHub Copilot** (`copilot -p "..."`): 適合快速程式碼建議與補全
  - 範例：`copilot -p "產生 SQL JOIN 的範例程式碼"`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/li195111) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
