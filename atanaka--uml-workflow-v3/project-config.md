---
trigger: always_on
description: 10-step UML workflow: scenario→activity→usecase→class→state→sequence→validation→security→code→test→traceability. Caching, resume, single-step modes.
---


# UML Workflow v3

完全統合されたtoken効率化UMLワークフロー。キャッシュ管理・段階的実行・UML(.uml)出力最適化をすべて自動化。

## 🎯 Overview / 概要

このスキルは3つの最適化機能を統合：

1. **キャッシュシステム（項目1）**: 中間成果物の自動キャッシュと再利用
2. **段階的実行（項目2）**: 必要なステップのみ実行
3. **UML(.uml)最適化（項目3）**: クラスモデルは常に2形式の `.uml`（OMG + Eclipse UML2）を生成。クラスモデル以外の `.uml` はデフォルトOFF（40%高速化）

**最大75%のtoken削減を実現**

---

## 🤖 EXECUTION INSTRUCTIONS FOR CLAUDE / Claude実行指示

**CRITICAL**: When a user requests to use this skill, Claude MUST follow this exact workflow.

### Trigger Patterns / トリガーパターン

Execute this workflow when user says:
- "uml-workflow-v3で〜を生成"
- "uml-workflow-v3を使って〜"
- "token効率化ワークフローで〜"
- Any mention of this skill by name

### Quick Execution Steps / クイック実行手順

```
0. Detect environment (PHASE 0.0) → resolve {SKILL_DIR}/{OUTPUT_DIR} and tool names
1. Determine project_name from user's request
2. Ask questions using the question tool (Web: ask_user_input_v0 / Local: AskUserQuestion)
3. Execute run_workflow.py via shell (Web: bash_tool / Local: Bash) with user's answers
4. Read execution plan JSON
5. Call each sub-skill in sequence
6. Present results (Web: present_files / Local: SendUserFile)
```

**Detail**: See CLAUDE_QUICK_GUIDE.md for concise reference, or follow detailed phases below.

### Execution Workflow (MUST FOLLOW) / 実行ワークフロー

Claude will execute this skill by following these phases in order:

0. **Environment Detection** - Detect Web vs Local, resolve {SKILL_DIR}/{OUTPUT_DIR} and tool names (PHASE 0.0)
1. **Initialization** - Set up Python environment
2. **User Dialogue** - Ask configuration questions using the question tool (Web: ask_user_input_v0 / Local: AskUserQuestion)
3. **Configuration** - Build execution config using Python scripts
4. **Execution Plan** - Display plan and get confirmation
5. **Step-by-Step Execution** - Execute each step with cache management
6. **Completion** - Present results and statistics

Each phase is detailed below with exact commands to execute.

### ⚠️ CRITICAL: 2-Phase Auto-Split Architecture / 2フェーズ自動分割

**Problem**: Running all 10 steps in a single conversation exhausts the context window (~200K tokens). Step 8 (code generation) requires reading the largest sub-skill PIPELINE.md (~14K tokens) plus generating substantial code output, which causes failures around Step 8.

**Solution**: The workflow automatically splits into 2 phases:

| Phase | Steps | Purpose | Context Usage |
|-------|-------|---------|---------------|
| **Phase A** | 1-7 | Modeling & Validation | ~80K tokens |
| **Phase B** | 8-10 | Code Gen, Test Gen, Traceability | ~60K tokens |

**AUTO-SPLIT RULES**:

1. **Full Workflow mode**: After completing Step 7, Claude MUST:
   - Present all Phase A outputs via `present_files`
   - Cache all outputs
   - Inform user: "Phase A (Modeling) complete. Please start a **new conversation** and say: `uml-workflow-v3で{project_name}のStep 8から再開`"
   - **STOP execution.** Do NOT proceed to Step 8 in the same conversation.

2. **Resume from Step 8+ mode**: Claude loads cached artifacts and executes Steps 8-10 in a fresh context.

3. **Models-only mode**: Steps 1-7 only — no split needed.

4. **Exception**: If user explicitly says "continue in this conversation" or "このまま続行", Claude may attempt Steps 8+ but should warn about potential context limits.

---

## 🌐 PHASE 0.0: Environment Detection & Mapping (MUST RUN FIRST) / 環境自動判定

**CRITICAL — このスキルは Web版 (Claude.ai Skills sandbox) と ローカル版 (Claude Code) の両方で動作する。**
他のどのフェーズよりも先に、必ず環境を判定してプレースホルダとツールを確定すること。

このドキュメント本文では、環境に依存するパスを次のプレースホルダで表記している。
PHASE 0.0 で実際の値に解決してから、以降のすべてのコマンドで置換して使うこと。

| プレースホルダ | 意味 | Web版 の値 | ローカル版 の値 |
|---|---|---|---|
| `{SKILL_DIR}` | スキル本体ディレクトリ | `/mnt/skills/user/uml-workflow-v3` | `~/.claude/skills/uml-workflow-v3` |
| `{OUTPUT_DIR}` | 成果物の出力先 | `/mnt/user-data/outputs` | 作業中のプロジェクトディレクトリ（CWD） |
| `{WORK_DIR}` | コード生成先のルート | `/home/claude` | 作業中のプロジェクトディレクトリ（CWD） |

### Step 0.0.1: 環境判定コマンドを実行

```bash
python3 - <<'PY'
import sys, os
# scripts ディレクトリを探す（Web → ローカルの順）
for cand in ("/mnt/skills/user/uml-workflow-v3/scripts",
             os.path.expanduser("~/.claude/skills/uml-workflow-v3/scripts")):
    if os.path.isdir(cand):
        sys.path.insert(0, cand); break
from env_paths import is_web_environment, get_skill_dir, get_output_dir
print("ENV        :", "WEB" if is_web_environment() else "LOCAL")
print("SKILL_DIR  :", get_skill_dir())
print("OUTPUT_DIR :", get_output_dir())
PY
```

- 出力の `SKILL_DIR` を `{SKILL_DIR}`、`OUTPUT_DIR` を `{OUTPUT_DIR}` および `{WORK_DIR}` として以降で使用する。
- ⚠️ **ローカル版では `{OUTPUT_DIR}` = カレントワーキングディレクトリ**。成果物を特定フォルダに出したい場合は、先に対象プロジェクトへ `cd` するか、環境変数 `UML_WORKFLOW_OUTPUT_DIR` を設定してから実行すること。

### Step 0.0.2: ツール対応表（環境で読み替える）

本文中の例は Web版のツール名で書かれている。**ローカル版 (Claude Code) では必ず右列のツールに読み替えること。**
特に質問ツールはこの読み替えを怠ると質問が出ずにスキップされる（XMI質問が出なかった既知不具合の原因）。

| 用途 | Web版 (本文の表記) | ローカル版 (Claude Code) |
|---|---|---|
| ユーザーへの質問 | `ask_user_input_v0` | **`AskUserQuestion`** |
| 成果物の提示 | `present_files` | **`SendUserFile`** |
| ファイル閲覧 | `view` | **`Read`** |
| シェル実行 | `bash_tool` | **`Bash`** |

> 💡 ローカル版で `AskUserQuestion` を使う際は、本文の選択肢（キャッシュ/実行モード/**XMI生成**/テスト生成/技術スタック）をそのまま options として提示すること。

---

## 🛡 PHASE 0.5: Resume Mode Prerequisite Check (CRITICAL)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atanaka/uml-workflow-v3](https://github.com/atanaka/uml-workflow-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
