---
trigger: always_on
description: 以 run_backbone_numpy_shared_trunk.py 為 golden，將 numpy trunk 對齊轉 Verilog；backbone 對拍 verilog_backbone2/；head 對拍以 verilog_head2/ 為準（verilog_head/ 僅舊版參考）
---


# Numpy Trunk → Verilog 對齊規則

本規則與 `verilog_rule.mdc`（語法／可合成底線）並存；若有衝突，**可合成與對拍語意**以本檔與 `verilog_rule.mdc` 較嚴者為準。

## 1. Golden 與產出位置

- **數學與流程依據**：`python/tracking/run_backbone_numpy_shared_trunk.py`（以下稱 **numpy trunk**）。
- **Verilog 實作目錄**：`python/lib/models/verilog/`（僅在此目錄新增／修改 RTL；ROM/RAM 若已在外部生成，介面以現有 top / testbench 為準）。
- **Verilog 目錄分工**（多條路徑並存，勿混淆）：
  - **`python/lib/models/verilog_backbone2/`** — **目前 backbone 對拍基準**（`TEST_backbone.v` 等）；與 numpy trunk backbone 階段 golden 對拍。
  - **`python/lib/models/verilog_head2/`** — **目前 head 對拍基準**：**不跑 backbone RTL**；以 numpy trunk 產出的**準確 activation**（例如 `backbone_after_norm_backbone_out_bi.txt`）串流進 `head_top`，驗證 conv → tail → `cal_bbox`；TB 為 `TEST_head.v`（編譯 `verilog_head2/*.v` + `Sram_tok1` 等）。
  - **`python/lib/models/verilog/`** — **端到端整鏈路**（`TEST.v`，暫不動）：`sglatrack_top` 含 backbone + head。
  - **`python/lib/models/verilog_head/`** — **舊版 head-only**（`Sram_sh1_lo`/`hi`、`mac_dv` 等）；**僅參考或比對舊實作，預設不作為對拍修改目標**。
  - 修改 RTL 時須先確認目標目錄：backbone → `verilog_backbone2/`；**head → `verilog_head2/`**（除非使用者明確指定 `verilog_head/` 或 `verilog/`）；全鏈路 → `verilog/`。
- **Activation 對答案檔**（Q8.8 二進位文字，一行一元素）：  
  `python/output/golden/vit_care_relu6_numpy_trunk_dim32_out/Activation/*.txt`  
  對應檔名多為 `*_bi.txt`（與腳本 `write_bi` 輸出一致）。RTL 除錯時以**此目錄**為 golden，不以「猜 golden」為準。
- **階段對拍義務**：詳見 **§21**（numpy trunk 寫出的 activation 須與 Verilog 同階段輸出對答案）。

## 2. 不硬體化、不當作 RTL 行為依據的程式碼

- numpy trunk 內僅負責 **輸出 activation / weight 的 `*_bi.txt`** 的路徑（例如 `write_bi`、`save_npy`、`write_wbi`、目錄 `_bi_act_dir` / `_bi_wgt_dir`）**不實作進晶片**，也不當作 datapath 規格；它們的用途是 **給 Verilog 模擬對答案** 與產生 ROM 初值檔。
- **主架構外**的輔助流程不強制硬體化：`argparse`、`main`、讀檔、`json`、`print`、`mkdir`、`cv2`（若有）等。

## 3. 可合成性

- 所有要進晶片（或 FPGA 正式 image）的 RTL **必須可合成**，並遵守 `verilog_rule.mdc`（例如 Verilog-2001、禁止用於「硬體邏輯」的 `initial` / `#delay` 等，依該檔全文為準）。
- 若為 **除錯專用**（例如僅模擬用的 `$display`、`ifdef` 區塊、不可綜合的暫時波形 dump），必須：
  - 以註解或 `ifdef` 標題**明寫「不可合成／僅 simulation」**，且
  - **註明為解決何種現象而插入**（例如：對拍哪一節點 golden、追查 argmax 錯位、SRAM 寫入下溢、sigmoid 全飽和等；見 **§10**），並
  - 預設關閉或易於關閉，避免誤綜。

## 4. 修改必須有依據

- 修改 Verilog **不可憑猜**：需註明或可追溯至 **numpy trunk 對應函式與行數**、**golden 檔名與 flatten 順序**、或**已約定的定點規格**（例如 Q8.8、`fp`/`linear` 與 RTL 的對應說明）。
- 若規格不明，應先補文件或先在 numpy／binary check 釐清，再改 RTL。

## 5. 語義不對齊時的優先順序（Python 先可硬體化）

- 若發現 numpy trunk 某段**不易硬體化**或與已定 RTL **語義不一致**，優先 **修改 `run_backbone_numpy_shared_trunk.py`**，使演算法改為：
  - 固定 shape、固定迴圈深度、
  - 明確的定點／截斷／飽和步驟（與未來 RTL 可一一對應），
  - 再更新 golden 與 RTL。
- 避免為了遷就錯的 Python 語意而去寫「無法對拍」的 RTL。

## 6. Verilog 改動幅度

- 以 **最小 diff** 為原則：盡量保留原有模組結構、介面與命名；**只改必要區段**解決對拍或合成問題。
- 非經使用者同意，不做與當前 bug 無關的大重構。

## 7. 訊號 ↔ Golden 檔名註解（強制）

- 在 `python/lib/models/verilog/` 內，凡會對應到 numpy trunk 中間輸出、且可用 Activation golden 對拍的 **模組輸出／暫存陣列／串流**，應在適當位置（模組頭或該訊號旁）加註解，格式建議：

  `// Golden: vit_care_relu6_numpy_trunk_dim32_out/Activation/<檔名>_bi.txt`

  若一訊號對應多檔或子路徑，寫清楚檔名與 tensor 對應關係（例如 token 順序、flatten 為 row-major 等，若 golden 有文件則引用）。

- 若某節點 numpy 有 `save_npy("xxx.npy")` 但 golden 實際為 `xxx_bi.txt`，註解以 **實際存在的 `Activation/` 檔名** 為準。

## 8. 硬體化範圍（排除清單）

- **不必**硬體化：僅供寫檔、寫 log、路徑組合、與 `np.load`/CLI 相關的程式碼（見第 2 節）。
- **必須**能對應到硬體 datapath 的：依 numpy trunk **主流程**會執行到的 `block_forward`、`attention_forward`、`layer_norm`、`linear`、`head_shared_trunk`、`conv2d`、以及（若納入範圍）`cal_bbox` 與 tracker 相關運算等；實作時以「可合成 + 可對拍」為取捨邊界。

## 9. 主架構優先

- 判斷「要不要硬體化」時，以 numpy trunk **正常 main 路徑**會跑到的運算為準；冷門分支、僅在缺少 golden 時才走的 fallback（例如自行算 adaptive）可標為軟體或第二階段，除非使用者明確要求納入 RTL。

## 10. 除錯環境（模擬不在本 repo）

- 假設 **Verilog simulation 在外部環境**執行；除錯可依賴：
  - 使用者**截圖**（波形／訊號），或
  - 在 RTL 暫加 **`$display`**（須標註僅 simulation、見 **§3**），由使用者截圖或貼 log 回傳數值。
- 凡為除錯而加入的 **`ifdef`／`$display` 區塊**，除 **§3** 外，**必須**在區塊上方或模組頭註解寫清：**為了解決／驗證什麼**（症狀或對拍目標）、**與哪個 golden 檔或哪一層節點相關**；避免僅寫「debug」而後續無法判斷是否可刪、與何問題綁定。細節見 **§10.1**（**註解義務**與 **`ifdef` 分區**兩條）。
- 若使用者**無法使用波形圖**（無 FSDB/VPD 或無法開 viewer），除錯方式以 **§10.1** 為準，不應假設能依波形縮小問題。
- Agent 修改 RTL 時應避免依賴「本機跑完 sim」作為唯一驗證，改以 **golden 檔 + 程式依據** 推進。
- 使用者實機之 **VCS 編譯／`./simv`** 流程見 **§22**。

### 10.1 僅能以 `$display` 除錯（無波形圖）

當環境**無法使用波形 viewer**、僅能靠仿真 log 中的 **`$display`／`$strobe`／`$fdisplay`** 觀察訊號時，建議遵守下列做法，以便與 **§16** 前向對拍且不致洗版：

- **取樣時機**：只在有意義的拍列印（例如 `mac_bp && mac_dv` 寫 SRAM 當拍、`done` 拉高當拍、`state` 切換後第一拍）；避免每個 `posedge` 盲目列印。
- **NBA 後數值**：要看 nonblocking 更新後的結果，優先用 **`$strobe`**，或在下一拍再讀已鎖存的 `reg`。
- **SRAM `CLK(~clk)`**：讀 `*_q` 時可改在 **`negedge clk`** 列印（與 macro 讀取邊沿一致），避免 posedge 上位址已變而 `Q` 仍舊的錯覺（與 `head_top` 內 `DUMP_HEAD_BBOX_SRAM_NEGEDGE` 類思路一致）。
- **格式**：定點以 **`%h` 為主**，並在註解或同一行標明 **Q8.8／Q0.8**；負值須明示 **signed 二補數** 解讀，勿將 `0xff..` 誤當無號大正數。
- **列印語言（強制）**：`$display`／`$strobe`／`$fdisplay`／`$fwrite`／`$monitor` 等**仿真 log 內可見字串必須為英文（ASCII）**；使用者執行 `.v` 的環境**不支援中文**，禁止在 format string 或訊息內使用中文（含簡繁）。說明性文字請寫在 `//` 註解，或 Agent 回覆給使用者；RTL／TB 內僅輸出英文標籤（例如 `PASS`、`FAIL`、`bbox_mismatch`、`golden=`）。
- **`ifdef` 分區**：除錯列印一律以 **`ifdef` 編譯開關**封裝（預設關閉），見 **§3**；可按模組拆成多個 macro（例如 `DUMP_HEAD_SIZE_SAT`），按需開啟。**每個 macro 對應的區塊**須有註解說明：**欲診斷的問題**（例如「追查 tail_size 寫入前 `mac_cl` 過大導致 sigmoid 全飽和」）、**建議搭配編譯選項**、以及**若問題已解決是否應移除或改為預設關閉**。
- **註解義務（`ifdef`／`$display` 區塊）**：凡為除錯插入的列印，**不可**只留開關名稱；至少應在 **`ifdef` 上一行區塊註解**或**模組頭對應小節**寫明：
  - **要解決／驗證的現象**（例如 bbox 與 golden 差、`sc_q` 全為下限、與某 `Activation/*_bi.txt` 第一個分歧層級等）；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
