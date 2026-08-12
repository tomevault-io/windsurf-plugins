---
trigger: always_on
description: Portable Verilog-2001 RTL rules (FSM, latch avoidance, unified if/else-if for seq+comb always, parallel-if split, cross-always multi-driven, ROM/SRAM read contracts). Copy to any project; chip-specific golden paths belong in separate rules.
---


# Verilog RTL 設計規範

**適用範圍**：可複製到任意 Verilog-2001 RTL 專案。全文以**可合成、可維護**為主；**§7 起**含常見 **TSMC 1P SRAM／ROM `CLK(~clk)`** 範例與除錯備忘——若你的 macro／供應商不同，保留**契約三行**與**ADDR/USE 兩段**思想，替換接腳與 latency 即可。晶片或專案專用的 golden 路徑、檔名對照請放在**另一份**規則，勿寫進本檔。

## 語言規則
- 僅使用 Verilog-2001 語法，禁止 SystemVerilog 特性（logic, always_ff, always_comb, interface, typedef, enum, struct, package, assertions）
- 所有 RTL 必須可合成（synthesizable），禁止 initial（硬體邏輯）、# delay、fork/join、force/release
- **`reg` 不可在 `always` 區塊內宣告**（含 `if` 內）；暫存器一律在 module 層級宣告

## 程式結構
- Parameter 宣告在前，reg 宣告居中，always block 在後
- 不交錯 reg 宣告與 always block（`assign`／`always` 之後不得再新增 `reg`；debug 用暫存器須與其他 `reg` 同區）
- 每個 always block 前必須有一行用途註解
- **時序與組合** `always` 一律遵守 **「`if` / `else if` 鏈（共通規範）」**；若同層曾有平行多個 `if`，依該節拆分或改寫，拆分後須排除 **跨 `always` 多驅動**；組合另須 **區塊頂端 default**，避免 latch

## 模組化設計
- 採用 top-down 架構：top → controller → datapath modules
- 介面必須定義 clk, reset, start, busy, done, data_in, data_out
- 大邏輯拆分為 controller / compute core / pipeline stage / memory interface

## FSM 設計
- FSM 必須獨立為三段式：state register / next-state logic / output logic
- 使用 parameter 定義狀態名稱
- **`state` 暫存** 與 **`next_state` 組合** 各一個 `always`；**datapath／控制輸出** 可拆成多個 `posedge` `always`（见下方 **「`if` / `else if` 鏈」** 與拆分節），但 **不可** 與 `state`／`next_state` 寫在同一 block

## 時序與組合邏輯
- `always @(posedge clk)` 用於 sequential（**時序 block**），`always @(*)` 用於 combinational（**組合 block**）
- 優先使用同步 reset
- Sequential 用 `<=`（non-blocking），combinational 用 `=`（blocking）
- 組合邏輯必須覆蓋所有輸出，**禁止 latch 推論**（細則见下節）
- **時序與組合 block 皆須**遵守下方 **「`if` / `else if` 鏈（共通規範）」**——不是只約束其中一種

### `if` / `else if` 鏈（時序與組合 always 共通，必守）

**適用範圍**：凡 `always @(posedge clk)`（時序）與 `always @(*)`（組合）內的條件分支，**同一套結構規範**；不可只對組合要求互斥、時序卻放任同層平行 `if`。

#### 同層結構（定義）

1. **同一巢狀層級**內，對該層的互斥／優先決策只能有一個開頭的 `if`；其後同層條件**必須**寫成 `else if`（需要時以 `else` 收尾）。
2. **巢狀**時：外層是一條 `if`／`else if` 鏈；進入某一分支後，**內層再各自**遵守「一個 `if` + 其後皆 `else if`」。不可在同一層平行寫：
   ```verilog
   // BAD (same nesting level): two sibling ifs
   if (cond_a) ...;
   if (cond_b) ...;   // prefer: else if (cond_b); split always ONLY if LHS sets differ
   ```
3. **`case`／`casex`** 與 `if` 鏈並存時：同一組輸出在「`case` 分支」與「區塊上方的 `if`」之間若會同拍／同拍組合覆寫，須合併為**單一優先鏈**或依拆分節移到不同 `always`（時序）／先 default 再單一覆寫路徑（組合）。

#### 為何時序也要遵守（勿誤讀成「時序可平行 if」）

| | 組合 `always @(*)` | 時序 `always @(posedge clk)` |
|--|-------------------|------------------------------|
| 同層平行多個 `if` | **禁止**（latch、多驅動、路徑不明） | **禁止**（同 reg 多段 NBA 時順序即語意；與 `case` 混寫易 GLS／X） |
| 合規寫法（預設） | 頂端 default + 單一 `if`／`else if` 鏈，或 `case`+`default` | **同一** `always` 內改成 `if`／`else if` 優先鏈 |
| 合規寫法（例外） | — | **僅當**各分支 LHS **互不相同**且須同拍並行更新 → **拆成多個** `always`（每 block 各自一條鏈） |

> 舊表述「時序平行 `if` 無 `else`＝enable FF 故允許」**僅**指「**單一** `if (en)` 更新、條件不成立則 hold」這種**一個** enable；**不表示**允許同層再並排第二個獨立 `if`。遇到同層 sibling `if`：**先**改成同一 block 的 `if`／`else if`；**只有** LHS 集合不同、又必須同拍都更新時，才拆 `always`（勿把本可互斥的條件硬拆，也勿對須並行的獨立 LHS 誤用 `else if` 串成互斥）。

#### 允許的「單一 enable」（時序專用，不是平行決策）

```verilog
// OK: one enable; hold when en==0  →  enable FF (not a comb latch)
always @(posedge clk) begin
    if (reset)
        foo <= 'd0;
    else if (en)
        foo <= next_foo;
end
```

```verilog
// OK: one-cycle pulse — DFF samples fire each cycle (fire already 1-cycle)
always @(posedge clk) begin
    if (reset)
        pulse <= 1'b0;
    else
        pulse <= fire;
end
```

> `fire` 須為組合／已對齊的單週期條件。**避免**「`else` 裡先 `pulse<=0` 再 `if (fire) pulse<=1`」靠同拍後寫覆蓋；若條件不只 `fire` 一項，再用 `if`／`else if`／`else` 展開（见下方脈衝模板）。
```verilog
// BAD: same level, sibling ifs (seq or comb) — do not leave like this
always @(posedge clk) begin
    if (reset) begin /* ... */ end
    else begin
        if (cond_a) /* drive set A */ ;
        if (cond_b) /* drive set B */ ;
        if (cond_c) /* drive set C */ ;
    end
end
```

對上例 **BAD** 的改法（順序固定）：

1. **預設**：A／B／C 改成**同一個** `always` 內的 `if`／`else if`／`else` 優先鏈（條件互斥或需排優先時皆然；LHS 有重疊時**必須**如此，**禁止**拆成多個 `always` 以免多驅動）。
2. **例外才拆**：僅當 A／B／C 的 LHS **互不相同**，且語意上須**同拍並行**更新（例如管線各級）——此時**不可**用 `else if` 強行互斥（會少更新某一級），應拆成**多個獨立時序 `always`**，每個內部仍是各自的 `if`／`else if` 鏈。

```verilog
// OK (default): same always, if / else if priority chain
always @(posedge clk) begin
    if (reset)
        cnt <= 'd0;
    else if (unit_done)
        cnt <= 'd0;
    else if (stream_en)
        cnt <= cnt + 1'b1;
end
```

```verilog
// OK (exception): LHS disjoint + must update in parallel → one always per LHS set
// Purpose: pipeline stage A
always @(posedge clk) begin
    if (reset) /* clear A */ ;
    else if (mac_valid) /* drive set A only */ ;
end
// Purpose: pipeline stage B
always @(posedge clk) begin
    if (reset) /* clear B */ ;
    else if (compute_valid) /* drive set B only */ ;
end
```
#### 組合 block 額外義務

- 仍須遵守 **「避免產生 latch」**：頂端 default、`case` 有 `default`、路徑賦值齊全。
- Port mux／多路請求：同層必須 `if`／`else if` 互斥（见 §8.4）；**禁止**兩個 `if` 可能同時成立。

### 避免產生 latch（組合邏輯完整性，必守）

綜合工具在 **`always @(*)`**（或對敏感列表內所有變化都會執行的組合 `always`）中，若某個被賦值的 `reg`／`wire` **並非每一條執行路徑都有賦值**，會推論 **latch** 以「記住上一拍值」。**一律禁止** 依賴此類 latch；須改寫為明確暫存器（`posedge clk`）或補齊賦值。

#### 常見成因（寫 code 時逐項自查）

| 問題 | 說明 |
|------|------|
| **`if` 無 `else`** | 條件不成立時該訊號無賦值 → latch |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
