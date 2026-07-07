---
trigger: always_on
description: <!-- managed-by: hamanpaul/paulsha-conventions@v1.0.10 -->
---

<!-- managed-by: hamanpaul/paulsha-conventions@v1.0.10 -->
<!-- CLAUDE.md 為單一事實來源；AGENTS.md / GEMINI.md / .github/copilot-instructions.md 為指向本檔的 symlink，只需維護本檔 -->
policy_version: 1.0.10
<!-- policy_version 為 policy_check R-14 machine-readable marker；需保持裸行格式，請勿移入 frontmatter 或 code block。 -->

# serialwrap — AI Agent Policy Checklist

本文件為所有 AI agent（Claude、GitHub Copilot、Gemini 等）在本 repo 工作時必須遵守的政策清單。

## 分支政策

- **禁止直接 commit 到 `main` 分支**，所有變更必須透過 PR。
- 跨多個子項目或長期功能開發，建議使用 `git worktree` 避免分支污染。
- 分支命名慣例：`feature/<issue-id>-<short-desc>`、`fix/<issue-id>-<short-desc>`。

## 變更紀錄政策

- 自 policy v1.0.9/v1.0.10 的 **fragment 硬切換**起改採 **per-PR changelog fragment** 模型（消除並行 agent 對 `[Unreleased]` 的衝突）：所有 production code 變更，**必須新增 `changelog.d/<issue>-<slug>.md` fragment**（`changelog.d/` 的 direct child、`.md` 結尾，非巢狀路徑）。R-09 驗本 PR 有無 direct fragment——`code_paths` 有變動而缺 fragment → **FAIL**（純文件／release PR 因不動 `code_paths` 不觸發）；例外走 `skip-changelog` label 並附理由。
- **release 收斂**：以 `python3 -m policy_check.changelog collate --version X.Y.Z --date YYYY-MM-DD` 把 `changelog.d/*.md` 依 type 收斂成 Keep a Changelog dated 段並清空目錄；R-04 於 v1.0.9+ 僅要求 `# Changelog` 標題、不再強制 `[Unreleased]`（本檔仍保留空 `[Unreleased]` 供人閱讀無妨）。
- 版本號更動時，同步更新 `VERSION` 檔案（`pyproject.toml` dynamic version 從 `VERSION` 讀）。

## 測試政策

- **完成任何 code change 前，必須執行**：
  ```bash
  python3 -m pytest -q tests/
  ```
- 亦可執行：
  ```bash
  python3 -m unittest discover -s tests -v
  ```
  （注意：unittest 不載入 `tests/conftest.py` 的強制 env 隔離與 live guard 防線，僅 state/WAL/events 維度有 per-file 隔離（`tests/state_iso.py`）；**有 production daemon 的機器一律以 pytest 為準**，#120。）
- pytest 下的隔離行為（#120）：`tests/conftest.py` 會硬覆寫 9 個 `SERIALWRAP_*` 目錄變數並 pop 5 個高優先變數（外層 shell export 無效），並於 suite 結束執行 live guard（state/WAL/config/daemon 四維快照比對；逃生閥 `SERIALWRAP_LIVE_GATE=warn`，warn 下結構性破壞仍 FAIL——warn 為明知風險的 opt-in：daemon TX/state 變更僅示警，開發者需自行確認；daemon 維度多層探測 system→user systemd→唯讀 RPC on-demand，全不可達才 SKIP，#121）。
- 既有失敗：`tests/test_multiagent_e2e.py::TestMultiAgentE2E::test_five_agents_three_rounds_no_conflict`（agent TX count mismatch，pre-existing）。
- 不得引入**新的**測試失敗。

## Policy Check 政策

- 完成任何 phase 前，必須執行：
  ```bash
  python3 -m policy_check --repo .
  ```
- policy engine pinned SHA：`ee87a6d5ed91209d944934a2559f4f2622fd1ac2`。
- 安裝命令：
  ```bash
  python3 -m pip install --user --disable-pip-version-check \
    "git+https://github.com/hamanpaul/paulsha-conventions.git@ee87a6d5ed91209d944934a2559f4f2622fd1ac2"
  ```

## Agent 檔案同步政策

- **`CLAUDE.md` 為唯一事實來源（single source of truth）**；只需維護本檔。
- `AGENTS.md`、`GEMINI.md`、`.github/copilot-instructions.md` 一律為指向 `CLAUDE.md` 的 **symlink**（相對路徑），不再各自維護內容；改 `CLAUDE.md` 即同步生效。
  - 合規性：policy_check R-13（`is_file()`）與 R-14（`read_text()` 找 `policy_version:`）皆會跟隨 symlink 解析到 `CLAUDE.md`，故四檔仍視為存在且版本對齊。
  - 若 symlink 遺失或被取代為一般檔，重建：
    ```bash
    ln -sf CLAUDE.md AGENTS.md
    ln -sf CLAUDE.md GEMINI.md
    ln -sf ../CLAUDE.md .github/copilot-instructions.md
    ```
- 本檔首行保留 `<!-- managed-by: hamanpaul/paulsha-conventions@v1.0.10 -->`，第 3 行保留裸行 `policy_version: 1.0.10`（R-14 machine-readable marker，勿移入 frontmatter 或 code block）。

## PR 政策

- 所有 PR 必須填寫 `.github/pull_request_template.md` 的 Policy Checklist（R-11）。
- PR checklist 項目：
  - [ ] 分支不是 `main`
  - [ ] 變更已記錄：新增 `changelog.d/<issue>-<slug>.md` fragment（code 變更必備；純文件／release PR 可改動 `CHANGELOG.md` 或免記）
  - [ ] `VERSION` 已更新（若有版本號變動）
  - [ ] `python3 -m pytest -q tests/` 通過（無新失敗）
  - [ ] `python3 -m policy_check --repo .` 通過
  - [ ] `CLAUDE.md` 已更新（`AGENTS.md` / `GEMINI.md` / `.github/copilot-instructions.md` 為 symlink，自動同步）
  - [ ] 已標記 exemption label（若適用）

## Exemption Label 白名單

以下 label 可豁免特定 policy 規則（需在 PR 標記）：

> 以下為 policy engine 實際認得的豁免 label（對齊 v1.0.10 引擎；名稱需精確，多為 `policy-exempt:*` 冒號式，R-09 為歷史命名 `skip-changelog`）。

| Label | 豁免項目 |
|-------|---------|
| `skip-changelog` | 免記 changelog fragment（R-09；純文件/CI 變更，附理由）|
| `policy-exempt:ci-tests` | 免 CI 執行測試（R-19）|
| `policy-exempt:issue-link` | 免 PR↔issue closing-keyword（R-17）|
| `policy-exempt:docs-sync` | 免 docs/README 對齊（R-18，WARN）|
| `policy-exempt:doc-reference` | 免 doc 懸空引用（R-22）|
| `release:<version>` | 免 VERSION↔最新 tag 一致（R-07；release PR 於 tag 建立前）|

## 語言政策

- 本 repo 文件、註解、docstring、README、規格、commit message 與 AI 回覆**一律使用繁體中文**。
- **例外（雙語／對外發布素材）**：
  - `README.md` 採**中英雙語並存**（`English` 章節 + `繁體中文` 章節；英文段為繁中段的對照翻譯，供非中文操作者閱讀）——兩段內容須保持一致，非以英文取代繁中。
  - `brag-output/**` 為**對外發布素材**（launch video composition brief 等，目標受眾與影片字幕皆為英文），以**英文**撰寫，不受「一律繁中」限制。此為刻意例外，範圍僅限該目錄。

## Commit 政策

- Commit message 使用 Conventional Commits 格式（繁中 subject）。
- 所有 AI-assisted commit 必須包含 trailer：
  ```
  Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
  ```

## 實際命令

> 開發/操作速查（由 `.github/copilot-instructions.md` 併入，並對齊現行 pipx + systemd + XDG 流程）。

執行期依賴：`PyYAML`；`pyserial` 為 **Windows 序列埠後端**（`sw_core/serial_port.py` 的 `_PySerialPort`，#84 PORT-1）依賴，`pyproject` 以 `sys_platform=='win32'` 條件安裝（Linux/WSL 預設走 termios 後端不需要；亦可手動裝以 `SERIALWRAP_SERIAL_BACKEND=pyserial` 覆寫）。human console 路徑另需 `jq`、`minicom`。套件以 `pyproject.toml`（setuptools）打包，console_scripts `serialwrap` / `serialwrapd`，內嵌資產在 `sw_core/assets/`。

```bash
# 安裝（pipx 隔離 venv + serialwrap setup；有 systemd → systemd-user，無 → on-demand 降級）
./install.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hamanpaul/serialwrap](https://github.com/hamanpaul/serialwrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
