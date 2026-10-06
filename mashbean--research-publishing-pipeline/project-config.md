---
trigger: always_on
description: 本目錄是研究文章自動生產線。當使用者要求寫文章、做研究、發稿時，請依照此指南操作。
---

# Research Publishing Pipeline — Claude Code 操作指南

本目錄是研究文章自動生產線。當使用者要求寫文章、做研究、發稿時，請依照此指南操作。

## 啟動條件

當使用者說以下任何一種話時，啟動 pipeline：
- 「寫一篇文章」「幫我寫 blog」
- 「研究 X 主題」
- 「啟動 article job」
- 提供了題目 + 來源 + 核心問題

## 模式判定（在 Step 0 之前先決定）

**預設行為（自 2026-05-03 起）= 學術／無 accent**。mashbean-accent 改為 opt-in，僅當 intake.yaml 顯式啟用時才套用。

intake.yaml 的兩個欄位決定走哪條路徑：

| `apply_mashbean_accent` | `core_question` 結構 | 模式 | Spec |
|---|---|---|---|
| `false` 或未設（**預設**） | ≥ 4 個獨立 sub-arg | **學術 + multi-agent**（無 accent） | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) + [specs/multi-agent-deep-research.md](specs/multi-agent-deep-research.md) |
| `false` 或未設（**預設**） | ≤ 3 個 sub-arg | **學術 + single-agent**（無 accent） | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) |
| `true` 或 `content_goal: personal_blog` | 任意 | **Blog（含 mashbean-accent）** | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) §「Opt-in」 |

**預設不啟用 mashbean-accent**：editor agent 不引入 lexicon、不寫自嘲、不寫括號 meta、不下跨領域比喻；`run_editorial_pass.py` 直接推進到 ready-to-publish。
**Opt-in accent 模式**：editor 與 writer 套用 mashbean-accent skill；editorial-pass 通過後推進到 accent-pending 等 accent subagent。
**Multi-agent 模式**會把 Step 3 deep research 拆成 5+1 個平行 subagent，整合工作由 main agent 直接做。

學術模式輸出預設發布到 **blog-pro**（pro.mashbean.net），accent 模式才發到 mashbean.net。

**Legacy 別名（向後相容）**：`formal_academic: true` / `content_goal: academic_paper` 仍被識別為「無 accent」信號（與預設一致，不再需要手動設定）。

## Pipeline 操作流程

### Step 0: 建立 Job

```bash
python3 scripts/start_article_job.py <job-id> \
  --title "..." --audience "..." --core-question "..." --thesis "..."
```

### Step 1: 論證架構驗證（P1 新增，人工確認後才往下）

在進入 deep research 之前，**必須先驗證核心論證的推理鏈是否成立**。

1. 根據 intake.yaml 中的 core_question 和 thesis，產出 1 頁的推理鏈草稿（可直接輸出到對話中，不需要存檔）
2. 推理鏈草稿必須包含：
   - 核心問題 → 3-5 個子論點 → 合成結論
   - **每個推理跳躍的明確標示**：「從 A 推論到 B，依據是什麼？這個推論是類比、歸納、還是因果？」
   - **最脆弱的假設**：整條推理鏈最可能被質疑的 1-2 個環節
   - **如果 thesis 涉及跨層級推論**（例如從國家層級推到個人層級、從一個場域推到所有場域），必須明確標示並論證為什麼推論成立
3. **暫停，等待使用者確認**推理鏈成立後，才進入 Step 2

這一步的目的是避免「研究了大量素材後才發現推理方向不對」的浪費。如果推理鏈在這一步就被質疑，應該在修正論證方向後再進入 deep research。

### Step 2: 自動推進到 Deep Research

```bash
python3 scripts/run_pipeline.py <job-id> auto
```

Pipeline 會自動：intake → scoped → researching，然後停下等 Deep Research。

### Step 3: Research（用 subagent）

#### 3a. Single-agent 流程（標準 blog）

啟動 research subagent：

```
Agent(
  description="Research for article <job-id>",
  subagent_type="general-purpose",
  prompt="你是研究代理。讀取 <job-dir>/prompts/agent-research.md 的完整指令，\n然後讀取 <job-dir>/intake.yaml 和 <job-dir>/deep-research-packet.yaml，\n執行研究任務並將結果寫入 <job-dir>/verification/ 和 <job-dir>/notes/。\n完成後執行：python3 scripts/run_deep_research.py <job-id> integrate"
)
```

或者，如果使用者已經從 ChatGPT/Claude Deep Research 拿到結果：
```bash
python3 scripts/run_deep_research.py <job-id> save-raw <file>
python3 scripts/run_deep_research.py <job-id> integrate
```

#### 3b. Multi-agent 流程（學術 + ≥4 sub-arg）

完整流程詳見 [specs/multi-agent-deep-research.md](specs/multi-agent-deep-research.md)。摘要：

1. **平行派 5 個 Opus sub-arg agents**（同一 message、平行 Agent calls）。每個 prompt 含領域、主張、必收 A 級來源 5-10 條、≥30 來源/≥10 A 級規模、輸出位置 `raw/sub-args/sub-arg-N-<topic>.md`。
2. **整合不派 subagent**——main agent 直接用 Bash 提取每份 sub-arg 的 § 1 主張重述 + § 待開發，然後 Write 一份 `raw/deep-research-output.md`，含 8 個 SECTION marker。歷史教訓：subagent 整合會 stall 在 600 秒 watchdog。
3. **可選：外部 cross-validation**：
   ```bash
   python3 scripts/run_external_deep_research.py <job-id>
   ```
   獨立 Anthropic API 路徑（Opus 4.7 + adaptive thinking + web_search）跑第二意見。
4. 最後跑 `python3 scripts/run_deep_research.py <job-id> integrate` 把 8 個 SECTION 拆到 notes/ + verification/。

整合檔的 `<!-- SECTION: rewrite_warnings -->` **必須** 列出 sub-arg agent 在文獻檢索後對主張的修訂版（如「pseudonymous vs real-name identification 區分」、「accountable pseudonymity 是政治經濟成就」等），否則下游 writer 容易回退到 intake.yaml 原版命題。

### Step 4: Writer（用 subagent）

啟動 writer subagent 寫研究草稿：

```
Agent(
  description="Write research draft for <job-id>",
  subagent_type="general-purpose",
  prompt="你是寫作代理。讀取 <job-dir>/prompts/agent-writer.md 的完整指令，\n然後讀取 <job-dir> 中的 intake.yaml、evidence-map.md、source-notes.md，\n產出 drafts/research-draft.md。語言：繁體中文。"
)
```

完成後推進狀態：
```bash
python3 scripts/update_job_state.py <job-id> --status drafted --last-deliverable drafts/research-draft.md
```

### Step 5: Critic（用 subagent）

```
Agent(
  description="Fact-check draft for <job-id>",
  subagent_type="general-purpose",
  prompt="你是查核代理。讀取 <job-dir>/prompts/agent-critic.md 的完整指令，\n然後讀取 <job-dir> 中的 drafts/research-draft.md 和 verification/evidence-map.md，\n產出 verification/fact-check-report.md。"
)
```

根據 critic 的 Decision：
- 「可進入 blog rewrite」→ `update_job_state.py --status fact-checking`
- 「需要補研究」→ 回到 Step 2

### Step 6: Writer rewrite（用 subagent）

```
Agent(
  description="Blog rewrite for <job-id>",
  subagent_type="general-purpose",
  prompt="你是寫作代理（改寫模式）。讀取 <job-dir>/prompts/agent-writer.md，\n讀取 research-draft.md 和 fact-check-report.md，\n產出 drafts/blog-rewrite.md。重點：轉成 blog 文體，套用所有禁則。"
)
```

```bash
python3 scripts/update_job_state.py <job-id> --status rewritten --last-deliverable drafts/blog-rewrite.md
```

### Step 7: Editor（用 subagent）

```
Agent(
  description="Editorial pass for <job-id>",
  subagent_type="general-purpose",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mashbean/research-publishing-pipeline](https://github.com/mashbean/research-publishing-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
