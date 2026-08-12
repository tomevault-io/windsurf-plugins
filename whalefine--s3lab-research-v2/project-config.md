---
trigger: always_on
description: - Python 到 RTL 轉換計劃過程中只實作 **test / inference** 流程，不處理任何 train 會用到的邏輯。
---


# Python 到 RTL 轉換計劃（CARE / test-only / post-embedding）

## 專案邊界

- Python 到 RTL 轉換計劃過程中只實作 **test / inference** 流程，不處理任何 train 會用到的邏輯。
- RTL 只處理 **一個 template + 一個 search** 的情況，不考慮 batch 與多 template / 多 search。
- RTL **不實作** `patch_embed` 與 `position embedding`；這兩部分保留在 Python / PyTorch 端。
- RTL 的輸入是 **已完成 patch embedding 與 position embedding 後的特徵 / token**。
- RTL 的輸入形式固定為 **分開的 template token 與 search token**，不可將兩者在 Python 端先合成單一 `320 token` 再當成唯一 RTL 輸入。
- RTL 的目標是從 post-embedding 特徵開始，跑完 **backbone + head + bbox 生成** 的 test 路徑。
- 若與 Python test 對齊到最終 bbox，需納入 `score_map / size_map / offset_map / cal_bbox` 等 test-time head 邏輯。

## 固定點格式規則

- 本專案 Python check、binary Python check、參數文字檔與 RTL 的**預設固定點格式一律使用 `Q8.8`**。
- `Q8.8` 定義：
  - signed fixed-point
  - 1 個 sign bit
  - 8 個整數位
  - 8 個小數位
  - 總位寬預設視為 16-bit signed
- 若某個 stage 暫存器、累加器、乘法輸出需要更寬位寬，必須另外明確標註，但最終對外 dump / 對拍格式仍優先回到 `Q8.8`。
- 沒有特別聲明時：
  - activation 使用 `Q8.8`
  - weight 使用 `Q8.8`
  - bias 使用 `Q8.8`
  - intermediate dump 使用 `Q8.8`
- 若未來要改成其他格式，例如 `Q4.12`、`Q16.8`，必須先同步更新：
  - Python check
  - binary Python check
  - `.txt` / binary `.txt` 轉檔規則
  - RTL module 介面與 testbench

## 固定研究主線

1. 使用 `vit_CARE_relu6.py` 訓練浮點模型。
2. 使用 `vit_CARE_relu6.py` 訓練出的 checkpoint，載入到 `vit_CARE_relu6_fixed.py` 做 test / 量化驗證。
3. 將該 checkpoint `.pth.tar` 轉成各個 parameter 的 `.npy`，給 Python check 使用。
4. 撰寫 Python check（讀 `.npy`），用 hardware-friendly 方式重寫 **backbone + head + bbox** 的 test 流程。
5. 將 `.npy` 轉成 `.txt` / binary text，給 RTL 讀取。
6. 撰寫 binary 版 Python check，完全模擬未來 Verilog 的 bit-width、截斷、捨入、除法近似與控制流程。
7. 將 binary Python check 對應轉成 Verilog RTL。

## 參考檔案與路徑

- 浮點訓練 backbone：`python/lib/models/sglatrack/vit_CARE_relu6.py`
- 定點測試 backbone：`python/lib/models/sglatrack/vit_CARE_relu6_fixed.py`
- 定點 test config：`python/experiments/sglatrack/vit_coco_uav123_care_relu6_fixed.yaml`
- checkpoint 匯出腳本：`python/tracking/export_checkpoint_npy.py`
- Python check 參考：`reference/rongxuan/05_PythonCheck/python_check.py`
- Verilog 前 binary Python check 參考：`reference/rongxuan/06_GetBinary/python_check_verilog.py`

## checkpoint 路徑規則

- `vit_CARE_relu6.py` 訓練出的 checkpoint 預設路徑為：
  - `python/output/checkpoints/train/sglatrack/vit_coco_uav123_care_relu6/sglatrack_ep0050.pth.tar`
- 之所以優先使用 `ep0050`，是因為 config 的 `TEST.EPOCH = 50`，test 預設即會載入第 50 epoch。
- 若之後改 config 名稱或 epoch，需同步更新：
  - 訓練 checkpoint 路徑
  - 匯出 NPY 的來源 checkpoint
  - Python check / binary check / RTL testbench 使用的 golden 版本

## patch embedding / position embedding 規則

- `patch_embed` 與 `pos_embed` 不進 RTL。
- Python 端必須先完成：
  - patch embedding
  - 加上 positional embedding
  - 保留 template token 與 search token 分開的輸出
- embedding 前段的 NPY dump 檔名固定為：
  - `template_after_patch_embed_out.npy`
  - `search_after_patch_embed_out.npy`
  - `template_pos_embed.npy`
  - `search_pos_embed.npy`
  - `template_after_pos_add_out.npy`
  - `search_after_pos_add_out.npy`
- 上述檔名定義如下：
  - `template_after_patch_embed_out.npy`：`template = patch_embed(template)` 之後的輸出
  - `search_after_patch_embed_out.npy`：`search = patch_embed(search)` 之後的輸出
  - `template_pos_embed.npy`：`self.pos_embed_z`
  - `search_pos_embed.npy`：`self.pos_embed_x`
  - `template_after_pos_add_out.npy`：`patch_embed(template) + pos_embed_z`
  - `search_after_pos_add_out.npy`：`patch_embed(search) + pos_embed_x`
- 硬體驗證的 golden input 應該是 **post-embedding tensor**，不是原始 RGB image。
- Python 端可做 `patch_embed(template) + pos_embed_z` 與 `patch_embed(search) + pos_embed_x`，但 **`combine_tokens` 必須保留在 RTL 端**。
- RTL 輸入固定定義為：
  - `template_post_embed`：shape `(1, 64, 768)`
  - `search_post_embed`：shape `(1, 256, 768)`
- 上述兩個 tensor 才是 RTL 測試與對拍的第一級 golden input；`320 token merged tensor` 只能作為 Python / RTL 中間節點 dump，不可取代 RTL 正式輸入介面。
- 若匯出 parameter 只為 RTL 使用，可排除：
  - `patch_embed.*`
  - `pos_embed*`
  - `cls_token`
  - 任何只屬於 embedding 前段的權重
- 若匯出 parameter 也要保留完整 Python 對照，可額外保存完整 checkpoint 對應 NPY，但 RTL 實際使用時只讀 post-embedding 之後需要的參數。

## test-only 流程規則

- 僅分析與實作 test path。
- 不閱讀、不修改、不 RTL 化以下 train-only 項目：
  - dataloader
  - data augmentation
  - loss function
  - optimizer
  - backward
  - scheduler
  - AMP
  - candidate elimination warm-up / train-only control
- Python / RTL 對齊時，優先以 `forward_test` 與 tracker `track()` 實際推論路徑為準。

## Golden 版本規則

- 本專案 golden 驗證採用 **軟體先產生 template/search post-embedding token，RTL 只驗證第二幀 model path** 的策略。
- 第一幀在 Python / PyTorch 端的用途是：
  - 讀入 `frame1`
  - 使用 `init_bbox` 建立 template crop
  - 完成 preprocess、`patch_embed`、`pos_embed_z`
  - 產生 `template_post_embed`
- 第二幀在 Python / PyTorch 端的用途是：
  - 讀入 `frame2`
  - 依第一幀初始化後的 state 建立 search crop
  - 完成 preprocess、`patch_embed`、`pos_embed_x`
  - 產生 `search_post_embed`
- RTL 的第一版不負責：
  - 第一幀初始化
  - image crop
  - preprocess / normalize
  - patch embedding
  - position embedding
- RTL 的第一版只負責從：
  - `template_post_embed`
  - `search_post_embed`
  開始，完成 `combine_tokens -> backbone -> head -> bbox`。
- 若日後要做 system-level RTL，才再把第一幀初始化、第二幀 search crop 與 tracker 後處理往前後延伸；第一版 golden 規則不預設納入。
- golden 版本至少要保存：
  - 第一幀的 `template_post_embed`
  - 第二幀的 `search_post_embed`
  - 第二幀的 `merged_tokens`
  - 第二幀的 block-level outputs
  - 第二幀的 `backbone_out`
  - 第二幀的 `score_map`
  - 第二幀的 `size_map`
  - 第二幀的 `offset_map`
  - 第二幀的 `pred_boxes`
  - 若有納入 tracker 後處理，再保存 `response_after_window`、`bbox_after_cal_bbox`、`final_bbox`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
