---
trigger: always_on
description: <!-- managed-by: hamanpaul/paulsha-conventions@v1.0.17 -->
---

<!-- managed-by: hamanpaul/paulsha-conventions@v1.0.17 -->
<!-- 此為 canonical 真檔；AGENTS.md / GEMINI.md / .github/copilot-instructions.md 為指向本檔的 symlink，只維護本檔 -->
policy_version: 1.0.17

# Agent Policy Checklist

本 repo 受 hamanpaul project policy v1.0.17 管轄。
所有 agent 進入 session 時，必須依下列 checklist 行動。

## 本 repo 的 profile
- policy_profile: `flat` （見 `.project-policy.yml`）
- policy_version: `1.0.17`

## 本 repo 定位
- `paulsha-cortex` 是治理平面拆包：persona 契約、coordinator 派工、control 檔案契約。
- 對 `paulsha-hippo` 維持零 runtime 依賴；僅 `persona/loader.py` 保留 upstream deck schema lazy import。
- path 契約以 `~/.agents/*` 與 `PSC_*` env 為主；service install 走 `cortex install service`。

## 動工前
- [ ] 確認當前分支不是 `main`
  - 若在 `main`，先開 `feature/<slug>` 分支
  - 若在 `feature/*`，可直接工作，或再開 `wt/<feature>/<subtask>`
- [ ] 若本任務跨多個子項，先建議用 `git worktree` 拆開

## 改 code 時
- [ ] **同一 PR 必須新增 `changelog.d/<slug>.md` fragment**（R-09 實際檢查的就是這個；
      只寫 `CHANGELOG.md [Unreleased]` 會 FAIL）
  - 檔名 slug＝分支名去掉 issue number，例：`feature/155-migrate-codex-relay-hook`
    → `changelog.d/migrate-codex-relay-hook.md`
  - fragment 必須 **commit** 才算（只 `git add` 不進 diff，R-09 仍 FAIL）
  - 發版時由 fragment 收攏進 `CHANGELOG.md`，因此並行 PR 不會在同一段互相衝突
- [ ] 另同步補 `CHANGELOG.md [Unreleased]` entry（repo 慣例，與 fragment 併存）
- [ ] 除非可明確標示為 docs-only / test-only / chore，否則不得省略 changelog
- [ ] code_paths 涵蓋的檔案變動皆視為 code change
      （本 repo 的 code_paths 含 `**/*.md`，故連改 `CLAUDE.md`／docs 也算）

## 改版號時（release 觸發時）
- [ ] 嚴格遵循 `<MAJOR>.<MINOR>.<PATCH>[-fix.N]`
- [ ] PATCH bump 對應 profile：
  - `stage-driven`: 一個 stage 落地
  - `flat`: 一個 feature batch 完成
- [ ] MINOR bump 需滿足：feature 群組全 landed + 7 天無 hotfix
- [ ] MAJOR bump 需使用者明確核可

## 完成任務（claim done）前
- [ ] `changelog.d/<slug>.md` fragment 已新增且已 commit（或 PR 標 `skip-changelog` + 理由）
- [ ] `CHANGELOG.md [Unreleased]` 有對應 entry
- [ ] `VERSION` 內容與意圖一致（release label PR 才可偏離 latest tag）
- [ ] PR body checklist 全勾（R-11；本 repo 目前無 PR template 檔案，
      checklist 由 PR body 自行帶出）
- [ ] **policy_check 必須帶 PR 上下文跑**，裸跑會給出假的 `fail: 0`：
      ```bash
      python3 -m policy_check --repo . \
        --pr-title "<PR 標題>" --pr-body "<PR body>" --pr-labels "<labels>" \
        --pr-base-ref main --pr-head-ref "feature/<slug>"
      ```
      CI 走 `.github/workflows/policy-check.yml` → conventions 的
      `reusable-policy-check.yml`，會傳這五個參數並啟用一批 PR／diff-aware 規則
      （含 R-09 的 diff 判定）；少了它們本機看不到那些 rule 的真實結果
- [ ] 若本 repo 有額外測試 / lint / build 指令，需一併通過
- [ ] 若跳過任何檢查，PR 必須帶對應豁免 label + 理由

## 禁止
- 直接 commit 到 `main`
- 建立不符合命名規則的分支（必須 `feature/<slug>` 或 `wt/<feature>/<subtask>`）
- 發明新 `policy-exempt:*` label（**只能用 policy 列舉的白名單**）
- 把 agent symlink（AGENTS.md / GEMINI.md / .github/copilot-instructions.md）還原成獨立複本（`agent_files.mode: symlink` 下 R-14 會 FAIL）

## Exemption Labels 白名單
僅允許使用以下 labels 豁免對應規則（其他一律視同未豁免）：
- `policy-exempt:readme-sections` — R-02 README 必備段落
- `policy-exempt:changelog-format` — R-04 CHANGELOG 格式
- `policy-exempt:pr-title` — R-10 PR title conventional-commit 格式
- `policy-exempt:branch-name` — R-12 分支來源規則
- `policy-exempt:agent-files` — R-13 agent convention files 存在
- `policy-exempt:cli-help` — R-16 CLI help 同步
- `skip-changelog` — R-09 code 變動要求 CHANGELOG entry（特殊用途，需附理由）
- `wip` — R-11 自動通過 PR body checkbox 未全勾（work in progress）

> 各 policy 版本新增的 exemption label 列於下方對應「新增規則」段。

## v1.0.1 新增規則（issue 連結 / docs 對齊 / 語言）
> 本段於 policy 1.0.1 隨 R-17 / R-18 與語言規範新增。

- **R-17（PR↔issue，FAIL gate）**：PR body 引用 issue（`#N`）時必須為 closing-keyword 形式（`Closes` / `Fixes` / `Resolves #N`），merge 由 GitHub 原生自動關閉 issue 並留下 cross-reference；只引用不關閉時上 `policy-exempt:issue-link`。
- **R-18（docs 對齊，WARN，不擋 merge）**：`code_paths` 有變動但 `README.md` / `docs/**` 未同步時提醒；純內部變動可上 `policy-exempt:docs-sync`。
- **語言規範（checklist）**：依 repo 來源決定語言——`github.com/hamanpaul/*`、`github.com/paulc-arc/*` → zh-tw；arcadyan GitLab → en_US。涵蓋 PR 標題／內文與所有 comment。本 repo 屬 `hamanpaul` → zh-tw。
- **動工前（軟性，不打斷流程）**：若任務對應某 issue，`gh issue view <N>` 核對相關性後分支可命名 `feature/<N>-<slug>`，開 PR 於 body 寫 `Closes #N`；查無對應 issue 照常進行，不另開、不停。
- **Exemption 白名單新增**：`policy-exempt:issue-link`（R-17）、`policy-exempt:docs-sync`（R-18）。

## v1.0.2 新增規則（CI 測試 / 版本同步）
> 本段於 policy 1.0.2 隨 R-19 / R-20 新增。

- **R-19（CI 必須跑測試，FAIL gate）**：repo 存在 `tests/`（含 `test_*.py` / `*_test.py`）時，`.github/workflows/**` 必須有至少一個 workflow 實際執行測試（pytest / unittest / npm test 等）；新增測試套件而 CI 未涵蓋時須同步補上；豁免 label `policy-exempt:ci-tests`。本 template 已內建 `tests.yml` 骨架（tests/ 出現即自動執行 pytest）。
- **R-20（workflow policy_version 同步，FAIL gate，無豁免 label，比照 R-14）**：workflow 內宣告的 `policy_version` / `POLICY_VERSION` semver 字面值必須與 `.project-policy.yml` 的 `policy_version` 一致；input 宣告與 `${{ inputs.* }}` 模板表達式不在檢查範圍。
- **Exemption 白名單新增**：`policy-exempt:ci-tests`（R-19）。R-20 不設豁免。

## v1.0.3–1.0.4 新增規則（機密掃描 tier）
> 本段於 policy 1.0.3 隨 R-21 與 `tier` 欄位新增，1.0.4 將標記 config 化。

- **R-21（機密掃描，FAIL gate）**：宣告 `tier: shareable` 的 repo 含雇主機敏標記（內部代號、裝置型號、build 主機等）、個人絕對路徑或憑證模式則 FAIL；`tier: work` / `personal` 視為 not-applicable；合法引用上 `.project-policy.yml` 的 `secret_scan.allow`，命中時上 `policy-exempt:secret-scan` 並附理由。1.0.4 將機敏標記改為 baseline 資料檔 + per-repo extend-only 疊加。
- **`.project-policy.yml` 新增**：`tier`（`shareable` / `work` / `personal`）與 `secret_scan.allow` / `secret_scan.markers` / `secret_scan.public_names`。
- **Exemption 白名單新增**：`policy-exempt:secret-scan`（R-21）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hamanpaul/paulsha-cortex](https://github.com/hamanpaul/paulsha-cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
