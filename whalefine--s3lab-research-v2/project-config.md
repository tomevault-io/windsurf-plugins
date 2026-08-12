---
trigger: always_on
description: 將reg修改成SRAM
---


# RTL → SRAM 改造建議（backbone + head + 整合版 verilog2）

> **規格**：Q8.8（16-bit signed），全專案共用。
> **規則依據**：[.cursor/rules/verilog_rule.mdc](../../.cursor/rules/verilog_rule.mdc)（**§7.7 SRAM 時序**、**§8 Reg→SRAM 檢查清單**）、[.cursor/rules/numpy-trunk-to-verilog.mdc](../../.cursor/rules/numpy-trunk-to-verilog.mdc)
> **判斷門檻**（全文件通用）：
>
> | 等級 | depth | 處理建議 |
> |------|-------|----------|
> | **必改 SRAM** | depth ≥ 1024（≥ 16 Kb / 2 KB 起跳） | 使用 1R1W 或單埠 SRAM macro |
> | **建議 SRAM** | 512 ≤ depth < 1024 | 視製程 macro 表決定，通常 ≥ 256 word 即可省面積 |
> | **可留 reg** | depth < 256 | flip-flop register file 即可 |
>
> 容量計算：**bits = depth × width**，**KB = bits / 8 / 1024**。

---

## 工作流總覽

| 目錄 | 角色 | SRAM 改造作為 |
|------|------|--------------|
| [python/lib/models/verilog_backbone2/](../lib/models/verilog_backbone2/) | **分模組除錯**：backbone 獨立驗證 | Phase 1：先在此改 SRAM port，獨立通過對拍 |
| [python/lib/models/verilog_head2/](../lib/models/verilog_head2/) | **分模組除錯**：head 獨立驗證 | Phase 2：先在此改 SRAM port，獨立通過對拍 |
| [python/lib/models/verilog2/](../lib/models/verilog2/) | **整合版**：backbone + head 端到端 | Phase 3：同步上述改動，加入 sglatrack_top 共享 SRAM mux |

> **本文件結構**：第一、二部分為 verilog_backbone2 / verilog_head2 的獨立分析（debug 階段參考），**第三部分為最終整合方案（verilog2，採用 Unified Shared SRAM Pool）**。

---

## 編譯開關與 RTL 雙路徑（`USE_REG_BUF` / `USE_SRAM_BUF`）

改 activation buffer 時，**必須**在 VCS 與 RTL 內維持兩條互斥路徑，方便 regression 與除錯：

| `+define` | 行為 |
|-----------|------|
| **`USE_REG_BUF`** | 保留 legacy **大 `reg` 陣列**（flip-flop register file）；對拍 baseline |
| **`USE_SRAM_BUF`** | 拆掉對應大 `reg`，改接 **SHC-SPMBSRAM compiler 產出的 macro**（見 §3.9 指令） |

**規則（必守）**：

1. **二選一**：禁止同時 `+define+USE_REG_BUF` 與 `+define+USE_SRAM_BUF`。 [TEST_backbone.v](../lib/models/verilog_backbone2/TEST_backbone.v) 若兩者皆定義會 `$finish`；兩者皆未定義時 TB 預設 `` `define USE_REG_BUF ``。
2. **RTL 用 `` `ifdef USE_REG_BUF `` / `` `ifndef USE_REG_BUF ``**（或對稱的 `USE_SRAM_BUF`）包住 reg 陣列與 SRAM mux／latency 邏輯；**同一模組內不可混用未 ifdef 的 `reg [15:0] buf [0:N-1]` 與 macro**。
3. **改完一步就對拍一步**：先確認 `USE_REG_BUF` 仍 PASS，再切 `USE_SRAM_BUF`。
4. **編譯列表示例**（macro `.v` 路徑依工作站；**不要**依賴 `*_wrap.v`）：
   ```bash
   # Reg baseline
   vcs verilog_backbone2/*.v memory/*.v \
     +define+TSMC_CM_NO_WARNING +define+USE_REG_BUF | tee runvcs.log

   # SRAM（含 compiler 產物；路徑依工作站）
   vcs verilog_backbone2/*.v memory/*.v \
     /path/Sram_tok1.v /path/Sram_tok2.v \
     /path/Sram_x.v /path/Sram_q.v /path/Sram_k.v /path/Sram_v.v /path/Sram_qkm.v \
     +define+TSMC_CM_NO_WARNING +define+USE_SRAM_BUF | tee runvcs.log
   ./simv | tee simv.log
   ```

> **注意**：`Sram_tok1` 與 `Sram_x` 為**不同 compiler 模組檔**（深度皆可用 12288×16，但 **不可** 只編譯一種檔名取代全部 instance）。instance 顆數見 **§1.0**。

---

# 第一部分：Backbone（`verilog_backbone2/`）— 分模組除錯版

> **目錄**：[python/lib/models/verilog_backbone2/](../lib/models/verilog_backbone2/)
> **常數**：`EMBED_DIM=32`、`NUM_HEADS=4`、`HEAD_DIM=8`、`N_TOKENS=320`、`MLP_DIM=128`
> **注意**：此版為**分模組除錯目錄**，`backbone_top` 內 **7 次** 呼叫 `transformer_block`（每 block 一組 `u_attn` macro）。整合到 `verilog2` 後為 **單一 `transformer_block` instance reuse 7 次**（見第三部分）。

## 1.0 Phase 1 實作快照（`USE_SRAM_BUF`，2026-05）

> 下列為 **目前 RTL 已落地** 之配置；與 **第三部分 Unified Pool（Phase 3）**、**§5.3 跳過 S_LOAD_X（Phase 3 目標）** 不同者已標 **(目標)**。

### 1.0.1 各檔 inline macro 顆數（`+define+USE_SRAM_BUF`）

| 模組 | macro 型號 | instance 名 | 顆數 | 有效深度×寬 | 對應 legacy reg |
|------|------------|-------------|------|-------------|-----------------|
| [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v) | `Sram_tok2` | `u_sram_tok2` | **1** | 10240×16 | `tok_buf`（block 間 ping-pong） |
| [backbone_top.v](../lib/models/verilog_backbone2/backbone_top.v) | `Sram_tok1` | `u_sram_tok1` | **1** | 10240×16 | `out_buf`（backbone norm 輸出） |
| [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v) | `Sram_tok1` | `u_sram_x` | **1** | 10240×16 | `x_buf`（主幹＋residual 錨點） |
| [transformer_block.v](../lib/models/verilog_backbone2/transformer_block.v) | `Sram_tok1` | `u_sram_tmp` | **1** | 10240×16 | `tmp_buf`（norm1/2、attn、mlp 暫存） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_x` | `u_sram_x` | **1** | 10240×16 | `x_in_buf`（norm1 快照；**與 `Sram_q` 不可共用**，見 §1.0.3） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_q` | `u_sram_q` | **1** | 10240×16 | `q_buf`（head-major） |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_k` | `u_sram_k` | **1** | 10240×16 | `k_buf` |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_v` | `u_sram_v` | **1** | 10240×16 | `v_buf` + **`ao_buf`（deviation：時間多工）** |
| [care_attention.v](../lib/models/verilog_backbone2/care_attention.v) | `Sram_qkm` | `u_sram_qkm` | **1** | 1280×16 | `qkm_buf` + `zr_buf`（時間多工） |
| [mlp.v](../lib/models/verilog_backbone2/mlp.v) | — | — | **0** | — | **無** 本地 10240 SRAM；FC1 讀 parent `u_sram_tmp` |

**單次 `backbone_top` 仿真編譯（macro `.v` 檔）**：`Sram_tok1`×2 檔案 + `Sram_tok2`×1 + `Sram_x`×1 + `Sram_q`×1 + `Sram_k`×1 + `Sram_v`×1 + `Sram_qkm`×1 = **7 種** compiler 產物。

**單次 `backbone_top` 仿真 instance 總數**（7 blocks）：`Sram_tok1`×(1+7×2)=**15**、`Sram_tok2`×1、`Sram_x`×7、`Sram_q/k/v/qkm`×7 各 = **15+1+7+28 = 51** 顆 macro instance（面積估算用；非 Phase 3 的 7 顆 pool）。

### 1.0.2 `mlp` 與 `transformer_block` 資料流（已實作）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
