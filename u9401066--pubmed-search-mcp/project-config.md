---
trigger: always_on
description: This document provides guidance for AI assistants working with the PubMed Search MCP server.
---

# GitHub Copilot Instructions for PubMed Search MCP

This document provides guidance for AI assistants working with the PubMed Search MCP server.

---

## ⚡ 開發環境規範 (CRITICAL)

### 套件管理：使用 UV (NOT pip)

本專案**必須**使用 [UV](https://github.com/astral-sh/uv) 管理所有 Python 依賴。
**所有命令（包括測試、lint、type check）一律透過 `uv run` 執行**，確保使用正確的虛擬環境與依賴版本。

> 💡 **UV 非常高效**：UV 使用 Rust 實作，比 pip 快 10-100 倍。即使是 `uv run pytest`，UV 也會在毫秒級確認環境一致後直接執行，幾乎零開銷。

```bash
# ❌ 禁止使用 (一律禁止直接呼叫，必須透過 uv run)
pip install <package>
python -m pytest
pytest
ruff check .
mypy src/

# ✅ 正確使用
uv add <package>           # 新增依賴
uv add --dev <package>     # 新增開發依賴
uv remove <package>        # 移除依賴
uv sync                    # 同步依賴
uv run pytest              # 透過 uv 執行測試（自動多核）
uv run python script.py    # 透過 uv 執行 Python
```

### 程式碼品質工具（全部透過 uv run 執行）

```bash
uv run ruff check .        # Lint 檢查
uv run ruff check . --fix  # Lint 自動修復
uv run ruff format .       # 格式化
uv run mypy src/ tests/    # 型別檢查（含 src 和 tests）
uv run pytest              # ⩡ 多核平行測試（預設 -n auto --timeout=60）
uv run pytest --cov        # 多核 + 覆蓋率
```

> ⚠️ **永遠不要**直接呼叫 `pytest`、`ruff`、`mypy`，一律使用 `uv run` 前綴。

### 🔒 Pre-commit Hooks (自動品質守門)

本專案使用 [pre-commit](https://pre-commit.com/) 在每次 commit 時自動執行品質檢查。

```bash
# 首次設定（uv sync 安裝依賴後）
uv run pre-commit install                       # 安裝 pre-commit hook
uv run pre-commit install --hook-type pre-push  # 安裝 pre-push hook

# 手動執行所有 hooks
uv run pre-commit run --all-files

# 更新 hook 版本（建議每月一次）
uv run pre-commit autoupdate
```

**Commit 階段自動檢查：**
- trailing-whitespace / end-of-file-fixer (自動修復)
- check-yaml / check-toml / check-json / check-ast
- check-added-large-files / check-merge-conflict / debug-statements / detect-private-key
- check-byte-order-marker / fix-byte-order-marker (自動修復)
- check-builtin-literals / check-case-conflict / check-docstring-first
- check-executables-have-shebangs / check-shebang-scripts-are-executable
- check-symlinks / destroyed-symlinks / check-vcs-permalinks
- check-illegal-windows-names / mixed-line-ending (自動修復)
- no-commit-to-branch (保護 main/master)
- name-tests-test (強制 test_*.py 命名)
- **ruff** lint (自動修復) + **ruff-format** (自動修復)
- **bandit** 安全掃描 (medium+ severity, `pyproject.toml [tool.bandit]`)
- **vulture** 死碼掃描 (`vulture_whitelist.py` 管理白名單)
- **deptry** 依賴衛生 (`pyproject.toml [tool.deptry]`)
- **semgrep** SAST 靜態安全分析 — **已移至 pre-push** (記憶體 300-500MB)
- **mypy** type check — **已移至 pre-push** (記憶體 500MB-1GB)
- **async-test-checker** async/sync 測試一致性 (`scripts/check_async_tests.py`)
- **file-hygiene** 檔案衛生檢查 (`scripts/hooks/check_file_hygiene.py`)
- **commit-size-guard** 限制每次 commit ≤30 檔案 (`scripts/hooks/check_commit_size.py`)
- **tool-count-sync** MCP 工具文檔同步 (`scripts/hooks/check_tool_sync.py`, 自動修復)
- **skills-frontmatter** 驗證 `.claude/skills/*/SKILL.md` 的 YAML frontmatter (`scripts/hooks/check_skills_frontmatter.py`)
- **evolution-cycle** 一致性驗證 (`scripts/hooks/check_evolution_cycle.py`)
- **future-annotations** 強制 `from __future__ import annotations` (`scripts/hooks/check_future_annotations.py`, 自動修復)
- **no-print-in-src** 禁止 src/ 使用 print() (`scripts/hooks/check_no_print.py`)
- **ddd-layer-imports** DDD 層級依賴檢查 (`scripts/hooks/check_ddd_layers.py`)
- **no-type-ignore-bare** 禁止裸 `# type: ignore` (`scripts/hooks/check_type_ignore.py`)
- **docstring-tools** MCP 工具必須有文檔字串 (`scripts/hooks/check_docstring_tools.py`)
- **no-env-inner-layers** 禁止內層 DDD 使用 os.environ (`scripts/hooks/check_env_config.py`)
- **source-counts-guard** 確保每來源 API 回傳量顯示 (`scripts/hooks/check_source_counts.py`)
- **todo-scanner** TODO/FIXME 掃描器 (警告, 不阻擋) (`scripts/hooks/check_todo_scanner.py`)

**Push 階段自動檢查：**
- **mypy** type check (`uv run mypy src/`, 記憶體 500MB-1GB)
- **semgrep** SAST 靜態安全分析 (`p/python` ruleset, 記憶體 300-500MB)
- **pytest** 全套測試 (`-n auto --timeout=60`)

```bash
# 跳過特定 hook
SKIP=mypy git commit -m "quick fix"
# 跳過所有 hooks（慎用）
git commit --no-verify -m "emergency fix"
```

### 🔄 自演化循環 (Self-Evolution Cycle - IMPORTANT)

本專案的 Instruction、Skill、Hook 形成一個自我演化的閉迴系統：

```
Instruction (copilot-instructions.md)
    │ 定義規範、引導 AI 使用 Skills
    ▼
Skill (SKILL.md 檔案)
    │ 確保建構完整、創建新 Hook
    ▼
Hook (.pre-commit-config.yaml + scripts/hooks/)
    │ 自動執行檢查、自動修正
    ▼
evolution-cycle hook (check_evolution_cycle.py)
    │ 驗證三者一致性、報告不同步處
    ▼
Feedback → 更新 Instruction & Skill → 循環完成
```

**新增 Hook 的完整流程：**
1. 創建 hook 腳本 → `scripts/hooks/<name>.py`
2. 註冊到 `.pre-commit-config.yaml`
3. 更新 `copilot-instructions.md` (Commit 階段自動檢查列表)
4. 更新 `git-precommit SKILL.md` (架構圖 + Hook 設定檔案表)
5. 更新 `CONTRIBUTING.md` (hooks 表格)
6. 執行 `uv run python scripts/hooks/check_evolution_cycle.py` 驗證

> ⚠️ 如果只做了 1-2 而沒有 3-5，evolution-cycle hook 會在下次 commit 時報錯。

**套件版本自動演化：**
```bash
uv run pre-commit autoupdate    # 更新 ruff、pre-commit-hooks 等版本
uv run pre-commit run --all-files  # 驗證更新後所有 hook 正常
```

### ⏱️ 測試執行時間 (IMPORTANT - 請務必閱讀)

本專案**強制**使用 **pytest-xdist** 多核平行測試（透過 `addopts = "-n auto --timeout=60"` 全局強制）。

```bash
# ✅ 所有測試命令自動帶 -n auto --timeout=60（不需手動加）
uv run pytest                # 多核執行（~67 秒）
uv run pytest tests/ -q      # 多核 + 簡潔輸出

# ✅ 導向檔案避免 terminal buffer 溢出
uv run pytest tests/ -q --no-header 2>&1 > scripts/_tmp/test_result.txt
# 等待 ~70 秒後再讀取結果

# ✅ 多核 + 覆蓋率（pytest-cov 完全支援 xdist）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [u9401066/pubmed-search-mcp](https://github.com/u9401066/pubmed-search-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
