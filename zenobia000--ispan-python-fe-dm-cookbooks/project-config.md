---
trigger: always_on
description: 本文件旨在建立一套涵蓋軟體開發全流程與 AGI 輔助技術的整體指引，幫助團隊與個人達成快速原型 (POC) 開發、知識積累與持續優化。文件整合了以下各部分內容：
---

# .cursorrules 文件：軟體開發與 AGI 融合指引 
 
本文件旨在建立一套涵蓋軟體開發全流程與 AGI 輔助技術的整體指引，幫助團隊與個人達成快速原型 (POC) 開發、知識積累與持續優化。文件整合了以下各部分內容： 
 
- 與使用者互動及內部記錄的基本指示   
- 開發過程中各角色的分工與職責   
- 軟體開發流程與運算思維方法   
- 快速 POC 開發的 SOP 指引   
- 持續學習與自我優化的機制   
- 套件與版本紀錄機制（含版本相依性守則）   
- 依賴查詢：透過官方文件 URL 搜尋套件相依性資訊 
 
--- 
 
## Windows CMD 操作指引

* **啟動命令提示字元**：按下 `Win` + `R`，輸入 `cmd`，然後按 `Enter`。
* **切換目錄**：使用 `cd`，例如 `cd C:\path\to\project`。
* **列出檔案與資料夾**：使用 `dir`。
* **建立資料夾**：使用 `mkdir 資料夾名稱`。
* **刪除檔案**：使用 `del 檔案名稱`。
* **刪除資料夾**：使用 `rmdir /S /Q 資料夾名稱`（包含子目錄且不提示）。
* **複製檔案**：使用 `copy 來源 目的地`。
* **移動檔案／資料夾**：使用 `move 來源 目的地`。
* **執行 Python 腳本**：使用 `python script.py` 或 `py script.py`。
* **串接多重指令**：

  * 使用 `&&` 僅當前一指令成功時才執行下一指令，例如：

    ```bat
    mkdir test && cd test
    ```
  * 使用 `||` 僅當前一指令失敗時執行下一指令，例如：

    ```bat
    type missing.txt || echo File not found
    ```
  * 使用 `&` 不論前一指令成功或失敗都執行下一指令，例如：

    ```bat
    echo step1 & echo step2
    ```

---

## Windows PowerShell 操作指引

PowerShell 為 Windows 上進階的殼層（shell）與指令腳本環境，與 Linux shell（bash/zsh）在語法與物件模型上有以下主要差異：

* **物件導向管線**：PowerShell 的管線傳遞物件，而非純文字。例如：

  ```powershell
  Get-Process | Where-Object { $_.CPU -gt 100 }
  ```
* **指令（cmdlet）命名慣例**：多為 `動詞-名詞` 格式，如 `Get-ChildItem`、`Set-Location`，對應 Linux 的 `ls`、`cd`。
* **變數宣告**：變數前需加 `$`，且不需顯式宣告類型，例如：

  ```powershell
  $files = Get-ChildItem -Path . -Filter *.txt
  ```
* **切換目錄**：使用 `Set-Location`（別名 `cd`）、`Push-Location` / `Pop-Location` 管理位置堆疊。
* **列出檔案**：使用 `Get-ChildItem`（別名 `ls`, `dir`）、可搭配 `-Recurse` 深度搜尋。
* **建立資料夾**：使用 `New-Item -ItemType Directory -Name 資料夾名稱`。
* **刪除檔案／資料夾**：使用 `Remove-Item 路徑`；加 `-Recurse` 刪除子目錄，加 `-Force` 忽略提示。
* **複製檔案**：使用 `Copy-Item來源 -Destination 目的地`。
* **移動檔案／資料夾**：使用 `Move-Item 來源 -Destination 目的地`。
* **執行 Python 腳本**：與 CMD 相同，使用 `python script.py`，但可在同一行中搭配 PowerShell 參數。
* **執行腳本設定**：預設禁止執行腳本，可透過 `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` 設定執行策略。

## 一、基本指示 (Instructions) 
 
- **可重用資訊記錄**   
  在與使用者互動過程中，若發現專案中可重用的資訊（例如函式庫版本、模型名稱、錯誤修正或收到的糾正），請立即記錄於本文件的 **Lessons** 區塊，避免日後重複相同錯誤。 
 
- **Scratchpad 作為思考與記錄工具**   
  - 使用本文件作為 Scratchpad（工作筆記區），組織與記錄所有新任務的思考、規劃與進度。   
  - 開發流程規劃 - 任務內容 
  - 接到新任務時，首先回顧 Scratchpad 內容，若有與當前任務無關的舊任務，請先清除。   
  - 說明任務內容、規劃完成任務所需步驟，可使用 todo markers 表示進度，如：   
    - [X] 任務 1   
    - [ ] 任務 2   
  - 完成子任務時更新進度，並於每個里程碑後反思與記錄，確保全局規劃與細節追蹤兼備。 
 
--- 
 
## 二、Cursor Learned 
 
- 處理搜尋結果時，確保正確處理不同國際查詢的字符編碼（UTF-8）。 
- 在 stderr 中輸出除錯資訊，同時保持 stdout 輸出整潔，便於整合管道操作。 
- 使用 matplotlib 畫圖時，若需採用 seaborn 風格，請使用 seaborn-v0_8 而非傳統 `seaborn`（因近期版本變更）。 
- 使用 OpenAI 的 GPT-4 時，請以 gpt-4o 作為模型名稱，尤其在具備視覺功能時。 
 
--- 
 
## 三、資料夾結構規劃 
 
/data_mining_course/
│
├── modules/
│   ├── module_01_eda_intro/                      # 模組1：課程導入與EDA複習
│   │   ├── slides/
│   │   │   └── 01_course_introduction.pptx
│   │   ├── notebooks/
│   │   │   ├── 01_pandas_basics.ipynb            # Pandas基礎操作複習
│   │   │   ├── 02_data_visualization.ipynb       # 資料視覺化技巧
│   │   │   └── 03_exploratory_analysis.ipynb     # 探索性分析方法
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_02_data_cleaning/                  # 模組2：資料清理與預處理複習
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_chunking_large_files.ipynb     # 大檔案分塊處理
│   │   │   ├── 02_handling_duplicates.ipynb      # 重複值處理
│   │   │   ├── 03_data_type_conversion.ipynb     # 資料型態轉換
│   │   │   └── 04_text_cleaning.ipynb            # 文字欄位清理
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_03_missing_outliers/               # 模組3：缺失值與異常值處理
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_missing_data_overview.ipynb    # 缺失值概述
│   │   │   ├── 02_imputation_methods.ipynb       # 各種插補方法比較
│   │   │   ├── 03_outlier_detection.ipynb        # 異常值檢測方法
│   │   │   └── 04_house_prices_case.ipynb        # House Prices資料集實作
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_04_categorical_encoding/           # 模組4：類別變數編碼方法
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_label_onehot_encoding.ipynb    # 標籤與獨熱編碼
│   │   │   ├── 02_count_frequency_encoding.ipynb # 計數與頻率編碼
│   │   │   ├── 03_target_encoding.ipynb          # 目標編碼
│   │   │   ├── 04_high_cardinality.ipynb         # 高基數特徵處理
│   │   │   └── 05_titanic_case.ipynb             # Titanic資料集實作
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_05_scaling_transformation/         # 模組5：特徵縮放與變數轉換
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_scaling_methods.ipynb          # 縮放方法比較
│   │   │   ├── 02_power_transformations.ipynb    # 冪轉換方法
│   │   │   ├── 03_outliers_impact.ipynb          # 異常值對縮放的影響
│   │   │   └── 04_insurance_case.ipynb           # 保險資料集實作
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_06_feature_creation/               # 模組6：特徵創造
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_interaction_features.ipynb     # 交互特徵創建
│   │   │   ├── 02_group_aggregations.ipynb       # 分組聚合特徵
│   │   │   ├── 03_time_derivatives.ipynb         # 時間衍生特徵
│   │   │   └── 04_nyc_taxi_case.ipynb            # NYC計程車資料集實作
│   │   ├── exercises/
│   │   └── resources/
│   │
│   ├── module_07_feature_selection/              # 模組7：特徵選擇與降維
│   │   ├── slides/
│   │   ├── notebooks/
│   │   │   ├── 01_filter_methods.ipynb           # 過濾法特徵選擇

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zenobia000) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
