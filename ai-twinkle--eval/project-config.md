---
trigger: always_on
description: > **本文件是所有開發者與 coding agent 的強制閱讀文件。**
---

# Twinkle Eval — 專案規範手冊（CLAUDE.md）

> **本文件是所有開發者與 coding agent 的強制閱讀文件。**
> 在修改任何程式碼之前，必須先完整閱讀本文件，確保所有變更符合本專案的設計理念與規範。
> 若提議的變更與本文件有所衝突，**必須先至 GitHub 開立 Issue 進行討論**，而非直接提交 PR 要求合入。

---

## 目錄

1. [專案定位與設計理念](#1-專案定位與設計理念)
2. [核心設計原則（不得違反）](#2-核心設計原則不得違反)
3. [架構總覽](#3-架構總覽)
4. [模組職責邊界](#4-模組職責邊界)
5. [擴充規範——如何正確新增功能](#5-擴充規範如何正確新增功能)
6. [新增評測 Benchmark 的完整規範](#6-新增評測-benchmark-的完整規範)
7. [必須先開 Issue 的情況](#7-必須先開-issue-的情況)
8. [程式碼風格規範](#8-程式碼風格規範)
9. [設定檔規範（config.yaml）](#9-設定檔規範configyaml)
10. [輸出格式規範](#10-輸出格式規範)
11. [依賴管理規範](#11-依賴管理規範)
12. [CLI 設計規範](#12-cli-設計規範)
13. [提交 PR 前的 Checklist](#13-提交-pr-前的-checklist)
14. [專案現況快照](#14-專案現況快照)
15. [貢獻者](#15-貢獻者)

---

## 1. 專案定位與設計理念

### 1.1 誕生背景

2025 年初，推理模型（reasoning model）開始大量出現。這類模型在輸出正式答案之前，會先產生大量的推理過程（chain-of-thought），導致每次 API 呼叫的回應時間遠高於傳統模型。

然而，當時現有的評測框架（如 iKala/ievals）採用**同步、逐題呼叫**的設計，面對推理模型時評測時間會等比例放大，一個完整的 benchmark 動輒耗費數小時。對於正在進行模型訓練迭代的團隊而言，這意味著每次訓練完都需要等待過久才能得到評測反饋，白白浪費 GPU 運行時間與開發週期。

**Twinkle Eval 因此而生**：以並行 API 請求為核心手段，讓評測速度不再是訓練迭代的瓶頸。實測相比 iKala/ievals 快 9–17 倍，使團隊能夠快速取得評測結果、驅動下一輪訓練決策。

### 1.2 核心設計哲學

**「輕量、單機、即裝即用」是這個專案的根本設計方向。**

本專案從一開始就以「`pip install twinkle-eval` 後，在單台機器上即可執行完整評測」為設計約束。這個約束是刻意的：

- **不依賴叢集基礎設施**：不需要 SLURM、Kubernetes、或任何分散式排程系統才能運作
- **不需要特殊硬體**：評測本身不需要 GPU，只需要能呼叫 API 的網路環境
- **降低使用門檻**：任何人在任何環境（本機、Colab、CI/CD）都能直接執行

多節點分散式評測（如 SLURM 支援）若作為 PR 提交，**定位是擴充功能（extension）**，不得影響單機執行路徑的正確性與簡潔性。核心程式碼必須在不依賴任何分散式元件的情況下完整運作。

### 1.3 關鍵設計邊界：本專案永遠不啟動模型端點

**Twinkle Eval 不負責啟動、部署、或管理任何 LLM 服務。**

本專案的職責範圍嚴格限定於：
> 「拿著評測題目，去呼叫**已經在外部運行的** API 端點，取得回答，並計算評測指標。」

使用者需要自行在外部啟動模型服務（vLLM、Ollama、OpenAI、NVIDIA Build 等），再將端點的 `base_url` 填入 `config.yaml`，Twinkle Eval 才開始工作。

這個邊界意味著：
- 本專案程式碼中**不得出現**任何啟動 `vllm serve`、`ollama run` 或其他模型服務的邏輯
- 本專案不管理模型的生命週期（啟動、關閉、重啟）
- 若 API 端點無回應，本專案的責任是報錯退出，而非嘗試修復或重啟服務

任何試圖在程式碼內部啟動模型服務的 PR，在未開 Issue 討論並取得 maintainer 明確同意前，不得合入。

### 1.4 核心目標

- **高效**：以並行請求大幅縮短評測時間，讓評測不再是訓練迭代的瓶頸
- **客觀**：透過選項隨機排列，消除模型對選項位置的偏好（參考 [Changing Answer Order Can Decrease MMLU Accuracy](https://arxiv.org/html/2406.19470v1)）
- **穩定性量化**：支援多次執行並計算標準差，反映模型一致性
- **易擴充**：模組化設計，讓新增 LLM 後端、評測策略、輸出格式都不需修改核心邏輯
- **API 相容優先**：以 OpenAI 相容 API 為統一介面，不綁定特定模型或服務商

### 1.5 本專案不是什麼

- 一個模型訓練框架
- 一個模型部署或服務管理工具
- 一個資料標注工具
- 一個需要叢集才能運作的分散式系統（單機是 first-class citizen）

**在新增任何功能之前，請先確認該功能是否符合上述定位與邊界。**

---

## 2. 核心設計原則（不得違反）

以下原則反映專案的根本設計決策，**任何違反這些原則的 PR 在開 Issue 討論並取得 maintainer 共識前，不得合入**。

### 原則 A：工廠模式 + 策略模式是唯一的擴充路徑

本專案使用三大工廠類別作為擴充點：

| 工廠 | 對應介面 | 負責建立 |
|------|----------|----------|
| `LLMFactory` | `LLM`（ABC） | LLM 後端實作 |
| `EvaluationStrategyFactory` | `EvaluationStrategy`（ABC） | 答案提取策略 |
| `ResultsExporterFactory` | （抽象基底） | 結果輸出格式 |

**規則**：
- 新增 LLM 後端 → 繼承 `LLM`，實作 `call()` 和 `validate_config()`，向 `LLMFactory` 註冊
- 新增評測策略 → 繼承 `EvaluationStrategy`，實作 `extract_answer()` 和 `get_strategy_name()`，向 `EvaluationStrategyFactory` 註冊
- 新增輸出格式 → 繼承對應基底類別，向 `ResultsExporterFactory` 註冊
- **禁止**在 `evaluators.py`、`main.py` 等核心流程中用 `if/elif` 判斷具體類型，應改用工廠或策略物件

### 原則 B：配置驅動（Config-Driven），禁止硬編碼行為

所有可變行為（評測方法、系統提示詞、資料集路徑、模型參數等）都必須透過 `config.yaml` 控制，不得在程式碼中硬編碼預設值或行為分支。

**禁止**：
```python
# 錯誤：硬編碼行為
if model_name == "gpt-4":
    do_something_special()
```

**允許**：
```python
# 正確：透過 config 控制
extra_body = model_config.get("extra_body", {})
```

### 原則 C：選項標籤不得硬編碼為 A/B/C/D

歷史上 A/B/C/D 被 hardcode，這已被識別為設計缺陷（PR #17 修正中）。未來所有涉及選項的邏輯必須動態偵測選項鍵，支援任意數量的選項：

```python
# 錯誤：硬編碼
for key in ["A", "B", "C", "D"]:
    ...

# 正確：動態偵測
option_keys = [k for k in question_data if k.isupper() and len(k) <= 2]
```

### 原則 D：評測結果必須確保不遺失

評測結果輸出時，**必須使用 append 模式**（`'a'`）而非 overwrite 模式（`'w'`）寫入 JSONL 檔案，以確保多檔、多 run 的結果都能正確累積。（此問題已記錄於 PR #17）

### 原則 E：API 金鑰絕對不得出現在輸出、日誌或 Git 歷史中

**兩條硬性規定，違反任一都是嚴重問題：**

1. **程式碼輸出**：在儲存結果或輸出日誌前，必須呼叫 `_prepare_config_for_saving()` 移除敏感資訊。任何新增的輸出路徑都必須遵守這個規則。

2. **Git commit**：含有真實 API 金鑰的 config 檔案**絕對不得 commit 到 Git**。
   - 本機測試用的 config 檔案（如 `config_test_*.yaml`、`config_local_*.yaml`、`config_*.local.yaml`）必須列入 `.gitignore`
   - **命名規範**：本機 config 檔案一律使用以下前綴或後綴之一，這些檔案已全局被 `.gitignore` 排除：
     - `config_local_*.yaml`
     - `config_test_*.yaml`（測試用）
     - `*.local.yaml`
   - 若不確定某個 config 是否含有 API 金鑰，在 commit 前先執行 `git diff --staged | grep -i "api_key"` 確認
   - coding agent 在建立任何含有 API 金鑰的 config 檔案時，**必須先確認該路徑已在 `.gitignore` 中**，然後才能寫入

### 原則 F：Backward Compatibility 優先

除非有充分理由並取得 maintainer 共識，新功能不得破壞：
- 現有 `config.yaml` 格式（舊設定檔應仍可正常運作）
- 現有 CLI 介面（現有命令列選項的行為不得改變）
- 現有輸出檔案格式（`results_{timestamp}.json` 和 `eval_results_{timestamp}_run{N}.jsonl` 的結構）

### 原則 G：本專案不啟動任何模型服務

本專案的職責是「呼叫已在外部運行的 API 端點」，程式碼中**絕對不得**出現：
- 以 subprocess、os.system 或任何方式執行 `vllm serve`、`ollama run`、`python -m ...` 等啟動模型服務的指令
- 監控、重啟、或管理外部模型服務進程的邏輯
- 假設自己有權限操作模型所在機器的任何邏輯

若 API 端點無回應或回傳錯誤，本專案的正確行為是：**依設定的 `max_retries` 重試後回報錯誤，絕不嘗試自行修復服務**。

### 原則 H：單機執行是 first-class citizen，分散式是可選擴充

本專案必須在**不依賴任何叢集基礎設施**的情況下完整運作。任何新增功能都不得讓「單機執行」的路徑變得更複雜或增加額外必要依賴。

分散式 / 多節點相關功能（如 SLURM 腳本）屬於擴充功能，應以**完全可選**的方式實作：
- 不引入新的 required dependency
- 不修改現有核心模組（`evaluators.py`、`main.py` 等）的主要邏輯
- 單機使用者若從未接觸分散式功能，應感受不到任何差異

---

## 3. 架構總覽

```
twinkle_eval/
├── __init__.py             # 套件入口，定義公開 API（__all__）
├── cli.py                  # CLI 入口點（entry point: twinkle-eval 命令）
├── main.py                 # TwinkleEvalRunner 主流程 + argparse 定義
│                           # ↑ 控制器層，協調各模組

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-twinkle/Eval](https://github.com/ai-twinkle/Eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
